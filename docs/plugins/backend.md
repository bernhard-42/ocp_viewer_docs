# The backend protocol

The viewer's Distance and Properties tools are answered by the [measurement backend](../concepts/backend.md), `ocp_viewer_core.backend.ViewerBackend`, from the exact geometry behind the mesh rather than from triangles. A kernel implements `BackendProtocol` for that: OCCT in `ocp_viewer_core.backend.OcctBackend` with `ocp_viewer_core/measure.py`, another kernel in its plugin, registered under its kernel tag. The backend calls it for every shape of that kernel, bodies, edges, vertices and helpers alike.

This page describes where the backend runs, what reaches it and what it answers, and then `BackendProtocol`, the part a plugin writes. ocp-viewer-smlibpy's [`backend.py`](https://github.com/bernhard-42/ocp-viewer-smlibpy) is the reference; [Toy plugins and testing](examples.md) has a minimal one.

## Where the backend runs

In every host the backend runs in another process than the user's script, so no object reaches it as a Python object and no import of the user's reaches it either:

| host | where the backend runs | how the model reaches it |
|---|---|---|
| OCP CAD Viewer for VS Code | `python -m ocp_vscode --backend --port <port>`, started by the extension with the configured interpreter | websocket to the extension, which relays it to the backend as it came |
| Jupyter CadQuery | the Jupyter server extension (`jupyter_cadquery/app.py`), one backend per viewer | HTTP post from the kernel |
| ocp-viewer | the viewer's server process (`ocp_viewer/server`) | websocket |
| build123d Studio | the sidecar's measure process | a local socket to the sidecar, a pipe to its measure process |

Three consequences shape the protocol:

- A body travels as bytes. The plugin's [`ShowProtocol.serialize`](show.md#serialize) turns it into bytes in the user's process, its `BackendProtocol.deserialize` rebuilds it in the backend's. The two processes can hold different versions of the plugin, so the bytes carry a format version.
- The plugin is found by an entry point. The backend process imports the plugin's measuring module by the entry point named like the kernel tag, the first time a model carries that tag, see [Registration and entry points](entry_points.md).
- The plugin must be installed in the environment the backend runs in. For VS Code that is the interpreter the extension starts the backend with, for Jupyter CadQuery the environment of the Jupyter server, which need not be the kernel's.

## What reaches the backend, and what it answers

### The model

After every show the core sends the backend `{"model": mapping}`, at one place for all hosts (`Viewer._show_impl` in `show.py`). The mapping is the tree ocp-tessellate's `tessellate_group` builds next to the mesh: groups with `id` and `parts`, and one leaf per measurable object.

```json
{"id": "/Group", "parts": [
  {"id": "/Group/box",    "shape": {"obj": "<base64 BREP>", "cache_id": "...", "name": "box"}, "loc": [[0, 0, 0], [0, 0, 0, 1]]},
  {"id": "/Group/edges",  "shape": ["<base64 BREP>", "<base64 BREP>"], "loc": [[0, 0, 0], [0, 0, 0, 1]]},
  {"id": "/Group/shape0", "shape": {"kernel": "smlib", "obj": "<base64 of serialize(shape)>"}, "loc": [[0, 0, 0], [0, 0, 0, 1]]}
]}
```

- An OCCT solid, face or shell leaf holds its instance (`obj` is the shape at its origin), an edge or vertex leaf a list of shapes. The codec (`codec.py`) writes OCCT shapes as base64 of OCCT's binary BREP.
- A kernel leaf holds `{"kernel": tag, "obj": shape}`, for bodies, edges, vertices and helpers alike. ocp-tessellate's `OcpObject.collect` emits it for every kernel object. Before anything is sent, `ocp_tessellate.kernels.encode_kernel_shapes` replaces `obj` by base64 of the plugin's `serialize(shape)`, so the codec never sees a kernel's object. So an OCCT leaf of edges travels as a list, a plugin's as one blob of bytes that holds them all.
- `loc` is the leaf's placement, from the locations of the groups above it and its own: `[[tx, ty, tz], [qx, qy, qz, qw]]`, a translation and a unit quaternion in the order of ocp-tessellate's `loc_to_tq`, or `null`. The point p of the body is drawn at R(q) p + t.

### The ids

The viewer selects sub-shapes by id, and the backend keeps one shape per id. For a leaf `{id}`:

| id | shape |
|---|---|
| `{id}` | the whole object: sent for a solid picked as a whole; the faces and shells of a shown face or shell are selected one by one |
| `{id}/faces/faces_{i}` | the face of index i |
| `{id}/edges/edges_{i}` | the edge of index i |
| `{id}/vertices/vertices_{i}` | the vertex of index i |

The viewer numbers faces and edges by the order of the tessellation's face blocks (`triangles_per_face`) and edge blocks (`segments_per_edge`), vertices by the order of `obj_vertices`. The backend must number them the same way, or a click reports another face than the one clicked: a plugin's `parts` returns the sub-shapes in exactly the order its tessellator lists them, seam edges included where the tessellator lists them. The two run in different processes, so the order has to be the same in every process: a kernel whose sub-shapes are ordered by a hash cannot meet this, and the interface has no way to carry explicit ids.

### The requests

The viewer reports every change of its state; the backend acts on two keys of a change set:

| key | value |
|---|---|
| `activeTool` | `"DistanceMeasurement"`, `"PropertiesMeasurement"` or `"None"` |
| `selectedShapeIDs` | `[id, shift]` for the properties tool, `[id1, id2, shift]` for the distance tool |

`shift` is whether the shift key was held. For the distance tool it asks for the distance of the two centers instead of the minimum distance (`center` below); the properties tool ignores it. A selection with another number of entries is not a request.

### The responses

The backend answers a request by returning a dict, which the host sends to the viewer. The backend adds `type: "backend_response"`, `subtype: "tool_response"` and `tool_type`; the rest is the core's, built from what the kernel's [`BackendProtocol`](#backendprotocol) gives, the same way for OCCT and every plugin (`measure.properties_response` and `measure.distance_response`). A plugin builds no response and formats no value.

A properties response:

| key | value |
|---|---|
| `shape_type` | what `properties` says (the panel translates Vertex, Edge, Face and Solid, and shows other names as they are) |
| `geom_type` | what `properties` says, with `SurfaceOfRevolution` shown as `Revolution` |
| `refpoint` | `[x, y, z]`, where the panel's marker points to |
| `result` | the panel's sections, a list of dicts |

A distance response:

| key | value |
|---|---|
| `result` | the panel's sections: the distance with `⇒ X | Y | Z` and `info` (`"min"` or `"center"`), the two points, and the angle where there is one |
| `refpoint1`, `refpoint2` | `[x, y, z]`, the ends of the distance arrow; `null` draws no arrow |

## `BackendProtocol`

Required of a plugin, since every leaf of a shown object has an entry in the model. `ocp_viewer_core.backend.OcctBackend` implements it for OCCT, so the backend treats every kernel alike: small functions, one for each step OCCT takes, while the core runs the sequence and makes the panel's rows. The shapes the methods take are what `deserialize` and `parts` returned: the plugin's own objects, never seen by the core. All eight methods are required: a plugin subclasses the protocol, so one it forgot fails with a `TypeError` when its object is created. It names its shapes as the protocol's parameter, so a type checker checks its methods against them; a plugin that names none works the same, unchecked. The core calls them by keyword, so the parameter names below are part of the interface.

```python
from ocp_viewer_core.backend import BackendProtocol, ShapeProperties, register_plugin

class SmlibBackend(BackendProtocol[Body | Face | Edge | Vertex | ShapeList]):
    def deserialize(self, data): ...
    def moved(self, shape, placement): ...
    def parts(self, shape): ...
    def kind(self, shape): ...
    def center(self, shape): ...
    def properties(self, shape): ...
    def closest_points(self, shape1, shape2): ...
    def closest_directions(self, shape1, shape2): ...
```

| method | what it does | OCCT's counterpart |
|---|---|---|
| `deserialize(data) -> shape` | the shape of the bytes of [`ShowProtocol.serialize`](show.md#serialize), in the backend's process: `deserialize(serialize(x))` is x again, which allows round trip tests | `ocp_utils.deserialize` of the BREP bytes `ocp_utils.serialize` wrote; an OCCT leaf of edges or vertices comes as a list of them |
| `moved(shape, placement) -> shape` | the shape moved by the leaf's `loc`, as a [`Placement`](show.md#get_location). Not called for a leaf without one | `shape.Moved(tq_to_loc(...))` |
| `parts(shape) -> (faces, edges, vertices)` | the shape's sub-shapes, in the order of [The ids](#the-ids). The vertices of a list of edges are the kernel's | `get_faces`, `get_edges`, `get_vertices` |
| `kind(shape) -> (shape_type, geom_type)` | what a shape is, see [`kind`](#kind). Asked at every click of the distance tool, so it must be cheap | `measure.occt_kind` |
| `center(shape) -> point \| None` | the point the distance tool's center mode measures from, see [`center`](#center). Asked at every such click, so it must not compute more than it needs | `measure.occt_center` |
| `properties(shape) -> ShapeProperties` | what the kernel knows about a shape, see [`ShapeProperties`](#shapeproperties). Asked only by the properties tool, so it may take its time | `measure.occt_properties` |
| `closest_points(shape1, shape2) -> (distance, point1, point2)` | the minimum distance of two shapes, and the points it is taken between | `measure.occt_closest_points` |
| `closest_directions(shape1, shape2) -> (direction1, direction2) \| None` | at the closest points of two edges or faces, the tangent of an edge or the normal of a face. `None` if there are no such directions. The core asks only for pairs of edges and faces | `measure.occt_closest_directions` |

In the backend's process the core runs, for every leaf: `deserialize`, then `moved` if the leaf has a placement, then `parts`. Points and directions are `(x, y, z)` tuples of plain floats.

**Keep a format version in the bytes.** The user's process and the backend's can have different versions of the plugin, as they can be different environments, and the interface does not look into the bytes. `ShowProtocol.serialize` writes its format's version first, and `deserialize` raises a `ValueError` for one it cannot read, which names the two environments as the thing to check. ocp-viewer-smlibpy's first byte is that version.

### `kind`

Two names. The shape's type: `"Vertex"`, `"Edge"`, `"Wire"`, `"Face"`, `"Shell"`, `"Solid"`, `"CompSolid"` or `"Compound"`. Its geometry: the names of `SURFACE_TYPES` and `CURVE_TYPES`, so the panel and the status line agree with the tessellation's `face_types` and `edge_types`, `"Point"` for a vertex, or `"Other"` for a body.

### `center`

Of a circle, ellipse, hyperbola or parabola its location, of a cylinder or cone face the point on its axis at the face's middle (the face's center of mass projected onto the axis, not the origin of the surface's own coordinate system, which can be anywhere on the infinite axis), of a sphere, torus or surface of revolution the origin of its axis, of any other edge its middle, of any other face or a body its center of mass, of a vertex its point. `None` if the shape has none, such as a degenerate edge: the panel then says `not measured: a shape without a center`.

### `ShapeProperties`

A `TypedDict` (`ocp_viewer_core.backend`) of plain numbers, the same for OCCT and every plugin. The core makes the panel's rows and strings of it, with `kind`: labels, order, sections, radius and diameter, supplements and the bounding box's center and size. One key is required:

| key | value |
|---|---|
| `refpoint` | where the panel's marker points to, `None` if there is no such point |

The rest is what the kernel has for the shape, and a key that is left out has no row:

| key | shape | row |
|---|---|---|
| `point` | vertex, required for one | `xyz` |
| `center` | edge, face | `center` |
| `radius` | edge | `radius / diam`, the value and its double |
| `radius` | face | `radius` |
| `major_radius`, `minor_radius` | edge | `major radius / diam`, `minor radius / diam` |
| `major_radius`, `minor_radius` | face | `major radius`, `minor radius` |
| `base_radius`, `half_angle` | face | `base radius`, `half angle`, the half angle in degrees |
| `offset` | edge | `offset` |
| `axis_location`, `axis_direction` | face | `axe loc`, `axe dir` |
| `start`, `vertex`, `middle`, `end` | edge | their names |
| `length` | edge, body | `length` |
| `area` | face, body | `area` |
| `volume` | body | `volume` |
| `tangent_start`, `tangent_end` | edge | `angle@0 to XY`, `angle@1 to XY`, the angle and its supplement |
| `normal` | face | `angle to XY`, the angle and its supplement |
| `bb_min`, `bb_max` | anything but a vertex, the two together | the section `bb`: min, center, max and size |

A kernel that cannot give a value for a degenerate shape leaves its key out, or gives `None` for `refpoint`: what does not need that value still answers. `ocp_viewer_core.measure.occt_properties` is the reference.

### How the distance tool uses them

The core takes the distance from `closest_points`, or in center mode the distance of the two `center`s. The angle of two edges or faces is the angle of the two `closest_directions`, for an edge and a face 90 minus that angle. The core names each direction from `kind`: `line` for a line, `tangent at P1` for another edge, `face normal` for a plane, `surface normal at P2` for another face. It does not ask for `properties`.

The backend measures two shapes of the same kernel only. A distance between shapes of two kernels (two plugins, or a plugin and OCCT) is answered with one row, `distance: not measured: shapes of two kernels (smlib, OCCT)`, and no arrow. To measure across kernels, the user converts to OCCT, see [Measuring objects of mixed kernels](mixed.md).

An exception in a plugin function while answering a request is caught and printed by the backend's error handler, and the request gets no answer; the backend keeps running. An exception while a leaf is loaded, a missing plugin too, is logged as `smlib shape '/Group/plate' not loaded: ...`, and costs only that leaf: the rest of the scene is measured, and a click on this leaf logs `unknown identifier received`.

## Registration

`register_plugin(KERNEL, SmlibBackend(), api=1)` registers the class when its module is imported, and the entry point of the group `ocp_viewer.kernels.backend` makes the backend's process import that module. [Registration and entry points](entry_points.md) describes both.

`plugins.backend_of(kernel)` looks in the registry first, and loads the entry point named like the kernel only when the kernel is not registered yet. A kernel neither registered nor declared raises a `LookupError` that names both ways and what is registered, which is what the log line of that leaf shows.

The backend's log says when a kernel's plugin is registered there, once per process, and names the kernel of every selected id:

```
Kernel plugin 'smlib' registered for measuring (ocp_viewer_smlibpy.backend)
smlib identifiers received '/Group/shape0/edges/edges_9', '/Group/shape0/edges/edges_1'
OCCT identifier received '/Group/box/faces/faces_0'
smlib/OCCT identifiers received '/Group/shape0/faces/faces_3', '/Group/box/faces/faces_0'
```

## Versioning

The kernel leaf is a new message shape between the core's two Python sides, the show and the backend, so both need a core that knows it: ocp-viewer-core with `ocp_viewer_core/plugins.py`, and ocp-tessellate 3.7 with the `kernel` tag. No JavaScript reads the backend's model: the core's JavaScript half never receives it, and VS Code's extension relays it as it came. The viewer's requests and responses keep their shape.
