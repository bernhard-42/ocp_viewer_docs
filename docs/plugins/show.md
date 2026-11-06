# The show protocol

`ShowProtocol` (`ocp_tessellate.plugin`) is what makes the viewer's own `show` and `show_all` take a kernel's objects. A plugin exports no show functions of its own, so everything a show does - the config, the camera, the tree, sending - is the viewer's, and a user's script looks the same for every kernel. For OCCT objects, `to_ocp`'s dispatch does what this protocol does for a plugin.

A plugin subclasses the protocol, so a required method it forgot fails with a `TypeError` when its object is created. A class that only has the methods, without subclassing, needs the optional ones too, since `register_show` tests for all of them. A plugin names the objects its methods take as the protocol's parameter, so a type checker checks them; a plugin that names none works the same, unchecked. The pipeline calls the methods by keyword, so the parameter names below are part of the interface:

```python
from ocp_tessellate.plugin import ShowProtocol

class SmlibShow(ShowProtocol[Body | Face | Edge | Wire | Vertex | Vector | Location | Plane | Axis]):
    def is_object(self, obj):
        return isinstance(obj, (_Body, Face, Edge, Wire, Vertex, Vector, Location, Plane, Axis))

    def convert(self, obj, name, color, alpha, helper_scale): ...

    def line(self, start, end):
        return Line(start, end)

    def serialize(self, shape): ...
```

| method | required | what it gives |
|---|---|---|
| [`is_object`](#is_object) | yes | whether one object is the kernel's |
| [`convert`](#convert) | yes | an atomic object as what the pipeline takes |
| [`line`](#line) | yes | the kernel's edge between two points |
| [`serialize`](#serialize) | yes | a shape as bytes for the measurement backend |
| [`get_children`](#get_children) | no | what a container of the kernel holds; without it every object is atomic |
| [`get_location`](#get_location) | no | an object's own placement; without it objects have none |

`register_show(KERNEL, tessellation, show, api=1)` takes this class and the [tessellation protocol](tessellation.md) when their module is imported, and an entry point makes the viewer import that module: see [Registration and entry points](entry_points.md).

## `is_object`

`is_object(obj) -> bool`

Whether one object is the kernel's: a body, face, edge, wire, vertex, vector, location, plane or axis, or a [container](#get_children) of the kernel. Lists, tuples and dicts are not its concern, because the viewer walks them and asks about each element; a `list` subclass or a `tuple` needs nothing else, in particular no build123d methods such as `first` or `filter_by`.

Test with `isinstance` on the kernel's own types, no duck typing: `is_object` runs in the process that holds the objects, so the types are there, and it decides who owns an object. Every installed plugin is asked in turn and the first to answer `True` owns the object, so an answer based on attributes could claim another kernel's objects. A raw OCCT shape is never asked about: it is OCCT's, whatever a plugin would answer.

It is asked about every variable of a scope at every debug stop, so it must be cheap. If it raises, the object counts as not the kernel's, and a warning says so once per plugin.

## `convert`

`convert(obj, name, color, alpha, helper_scale) -> OcpObject | CoordSystem | CoordAxis`

One atomic object of the kernel, or a location to draw, into what the pipeline takes. An `OcpObject` holds the kernel's shape, and from there it goes the way an OCCT object goes: the pipeline hands that shape to the [tessellation protocol](tessellation.md). Lists and dicts are not its concern, and neither are the kernel's own containers: those are for [`get_children`](#get_children), so anything but these three classes, an `OcpGroup` or `None` too, raises a `TypeError` that names the plugin's class. So does a solid, shell or face holding a list.

```python
from ocp_tessellate.plugin import CoordAxis, CoordSystem, OcpObject, object_color
```

| the kernel's object | what `convert` returns |
|---|---|
| solid | `OcpObject("solid", body, name=name, color=object_color(body, color, alpha))` |
| shell, sheet | `OcpObject("shell", body, ...)` |
| sketch, face | `OcpObject("face", body, ...)`, drawn on both sides (`renderback`) |
| edge, wire | `OcpObject("edge", [edge, ...], ...)`, always a list |
| vertex, point | `OcpObject("vertex", [vertex], ...)`, always a list |
| location, plane | `CoordSystem(name, origin, x_dir, z_dir, size=helper_scale, like=obj)` |
| axis | `CoordAxis(name, origin, direction, color, size=helper_scale, like=obj)` |

- The pipeline gives the object what `unify` gives an OCCT object: its instance through `cache_id`, its material and states, and the host's default color and width of the kind.
- `object_color(obj, color, alpha)` is the given color, else the object's own `color` attribute, else `None` for that default, which the pipeline gives the `alpha`. `color` takes what `Color()` takes: a web color name, `#rrggbb`, a tuple, a `Color`.
- `like=obj` is the same parameter OCCT's handlers use: `CoordSystem` and `CoordAxis` find the plugin of the object they depict and draw with its [`line`](#line), so they are the same classes and the same points as for OCCT, in the kernel's edges. `origin`, `x_dir`, `z_dir` and `direction` are `(x, y, z)` tuples.
- `helper_scale` is the absolute size the pipeline resolved. `convert` is the only place that needs it.

## `line`

`line(start, end) -> edge`

The kernel's edge between two `(x, y, z)` points, which every kernel can do. `is_object` must answer `True` for it, since the helper's edges go through the pipeline like any other.

## `serialize`

`serialize(shape) -> bytes`

What a leaf of the backend's model holds, a shape or a list of edges or vertices, as bytes for [`BackendProtocol.deserialize`](backend.md#backendprotocol), which reads them in the backend's process. Exact, and complete, since the backend has nothing else. Keep a format version in the bytes, see [`BackendProtocol`](backend.md#backendprotocol). It is called for every leaf before anything is sent, so what it raises makes the show raise, with the viewer and the backend still on the scene they had.

## `get_children`

`get_children(obj) -> list[(name, child, relative_placement)] | None`, optional

A kernel's own container: an assembly, or anything else that holds objects and is not a list, tuple or dict. `get_children` says what it holds, and `to_ocp` builds the tree with the walker it uses for lists and dicts. So the plugin builds no groups, and a child can be anything `show` takes: another container, a list or dict, an OCCT shape or a build123d object. `is_object` answers `True` for a container, and `convert` is not called for it. A container that `is_object` does not answer for is not drawn.

```python
class MyShow(ShowProtocol):
    ...
    def get_children(self, obj):
        if isinstance(obj, Assembly):
            return [(child.name, child.shape, child.relative_placement) for child in obj.objects()]
        return None
```

- `name` is the child's name in the tree. Animation tracks select objects by their tree path, so it should not depend on the order of the children. With `None` the child takes its own name or is named by its type, and repeated names are numbered as the viewer asks for.
- `relative_placement` places the child in the container, see [`get_location`](#get_location), or is `None`.
- `None` instead of a list means the object is atomic, and `convert` draws it. An empty list is an empty container, shown like an empty list.
- An entry that is not a `(name, child, relative_placement)` triple raises a `TypeError` that names the plugin's class and the entry.
- `get_children` is called more than once per show, so it must have no side effects. An error it raises is not hidden.

## `get_location`

`get_location(obj) -> Placement | None`, optional

The object's own placement, for a container or an atomic object. For an atomic object, `convert` draws the geometry in its local frame and the placement moves it there.

A `Placement` (`ocp_tessellate.plugin`) is a `NamedTuple` of `origin` `(x, y, z)` and `quaternion` `(x, y, z, w)`, so the plugin needs no OCCT location. It is the one type of location in the tree of `OcpObject`s and `OcpGroup`s, for every kernel, OCCT's too: `get_children` and `get_location` return a `Placement` or `None`, and anything else, a plain tuple too, raises a `TypeError`. A placement without 3 and 4 numbers, or whose quaternion is not of unit length, raises a `ValueError`.

| `Placement` | what it does |
|---|---|
| `Placement(origin, quaternion)` | builds one from its numbers |
| `Placement.from_axes(origin, x_dir, z_dir)` | builds one for a kernel without quaternions |
| `Placement.from_occt(loc)` | converts an OCCT `TopLoc_Location` |
| `a.mul(b)` | `a * b`: `b` in the frame of `a` |
| `p.inverted()` | the inverse of `p` |
| `p.is_identity()` | whether `p` does not move anything |
| `p.matrix()` | the 4 x 4 matrix of `p`, a float64 numpy array, for a kernel that places objects by one |

A child is placed at `relative_placement * get_location(child)`, so how a kernel stores its assembly stays inside the plugin:

| the kernel stores | `relative_placement` | `get_location` |
|---|---|---|
| geometry already moved into place | `None` | `None` |
| the parent holds `(child, placement)` | that placement | `None`, the root's own for a root |
| every object holds its placement relative to its parent | `None` | that placement |
| every object holds its absolute placement | `None` | the placement relative to its parent: `parent.inverted().mul(absolute)` |

The placements must be relative because of how the viewer works: an object's location is relative to the group above it, and an animation track adds to it, so a group turns about its own origin. Geometry that is already moved has no origin of its own and turns about the world origin. The viewer takes a position and a quaternion only, so a location with scale or mirroring has to be baked into the geometry.

## What the viewer then does

- `show`, `show_object` and `show_objects` take the kernel's objects next to any others. `to_ocp` converts them, directly and inside lists, dicts and containers. `names`, `colors`, `alphas`, `materials`, `modes` and every keyword apply: `reset_camera`, `collapse`, `helper_scale`, `transparent`, ... , and so do `set_defaults`. An object's own `name` or `label` attribute names it when no name is given, as for OCCT objects.
- `show_all`, and with it visual debugging, finds them in a scope through `ocp_tessellate.kernels.is_external_object(obj)`, under their variable names.
