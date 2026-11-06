# Toy plugins and testing

Two complete minimal plugins: one that draws, one that measures. They register themselves in the script that uses them, so they need no entry points; a plugin to install declares them, see [Registration and entry points](entry_points.md). The first imports `show` from `ocp_viewer_core.viewer`, which picks a host, so it runs where one is installed.

## A plugin that draws: a kernel of triangles

A kernel whose only body is a triangle, with all of [`TessellationProtocol`](tessellation.md) and [`ShowProtocol`](show.md), and a [`BackendProtocol`](backend.md) that measures next to nothing, in place. Its location is the same `CoordSystem` OCCT objects get, drawn with the kernel's lines.

```python
import json

import numpy as np
from ocp_tessellate.ocp_utils import BoundingBox
from ocp_tessellate.plugin import (
    CURVE_TYPES, SURFACE_TYPES, CoordSystem, OcpObject, Placement, ShowProtocol, TessellationProtocol, object_color,
    register_show,
)
from ocp_viewer_core.backend import BackendProtocol, register_plugin
from ocp_viewer_core.viewer import show

KERNEL = "triangle"

class Triangle:  # the kernel's one body
    def __init__(self, size=1.0):
        self.size = size

class Segment:  # the kernel's one edge
    def __init__(self, start, end):
        self.start, self.end = start, end

class Frame:  # the kernel's location to draw
    def __init__(self, origin):
        self.origin = origin

class Assembly:  # the kernel's one container: it holds its children with their places
    def __init__(self, links, location=None):
        self.links, self.location = links, location

class ToyTessellation(TessellationProtocol):
    def tessellate(self, shape, deviation, quality, angular_tolerance, compute_edges=True,
                   compute_uvs=False, normalize_uvs=True, shape_id="", debug=False):
        s = shape.size
        return {
            "vertices": np.array([0, 0, 0, s, 0, 0, 0, s, 0], dtype=np.float32),
            "triangles": np.array([0, 1, 2], dtype=np.int32),
            "normals": np.array([0, 0, 1] * 3, dtype=np.float32),
            "edges": np.array([0, 0, 0, s, 0, 0, s, 0, 0, 0, s, 0, 0, s, 0, 0, 0, 0], dtype=np.float32),
            "obj_vertices": np.array([0, 0, 0, s, 0, 0, 0, s, 0], dtype=np.float32),
            "face_types": np.array([SURFACE_TYPES["Plane"]], dtype=np.int32),
            "edge_types": np.array([CURVE_TYPES["Line"]] * 3, dtype=np.int32),
            "triangles_per_face": np.array([1], dtype=np.int32),
            "segments_per_edge": np.array([1, 1, 1], dtype=np.int32),
        }

    def discretize_edges(self, edges, deflection=0.1, shape_id=""):
        pts = np.array([(e.start, e.end) for e in edges], dtype=np.float32)
        return {
            "edges": pts.ravel(),
            "segments_per_edge": np.ones(len(pts), dtype=np.int32),
            "edge_types": np.full(len(pts), CURVE_TYPES["Line"], dtype=np.int32),
            "obj_vertices": pts.reshape(-1, 3).ravel(),
        }

    def convert_vertices(self, vertices, shape_id=""):  # the toy has no vertices of its own
        return {"obj_vertices": np.zeros(0, dtype=np.float32)}

    def bounding_box(self, shape):  # a frame has no extent, and raises
        if isinstance(shape, Triangle):
            points = [(0, 0, 0), (shape.size, shape.size, 0)]
        else:
            points = [p for e in shape for p in (e.start, e.end)]
        lo, hi = np.min(points, axis=0), np.max(points, axis=0)
        return BoundingBox({"xmin": lo[0], "xmax": hi[0], "ymin": lo[1], "ymax": hi[1], "zmin": lo[2], "zmax": hi[2]})

    def cache_id(self, shape):  # the geometry, not the object
        return f"triangle-{shape.size}"

class ToyShow(ShowProtocol):
    def is_object(self, obj):
        return isinstance(obj, (Triangle, Segment, Frame, Assembly))

    def line(self, start, end):
        return Segment(start, end)

    def serialize(self, shape):
        if isinstance(shape, Triangle):
            return json.dumps({"size": shape.size}).encode()
        return json.dumps({"segments": [(e.start, e.end) for e in shape]}).encode()

    def convert(self, obj, name, color, alpha, helper_scale):
        if isinstance(obj, Frame):
            return CoordSystem(name, obj.origin, (1, 0, 0), (0, 0, 1), size=helper_scale, like=obj)
        kind, shape = ("face", obj) if isinstance(obj, Triangle) else ("edge", [obj])
        return OcpObject(kind, shape, name=name, color=object_color(obj, color, alpha))

    def get_children(self, obj):
        return obj.links if isinstance(obj, Assembly) else None

    def get_location(self, obj):
        return obj.location if isinstance(obj, Assembly) else None

class ToyBackend(BackendProtocol):  # knows nothing but what a triangle's size says
    def deserialize(self, data):
        values = json.loads(data)
        if "size" in values:
            return Triangle(values["size"])
        return [Segment(tuple(a), tuple(b)) for a, b in values["segments"]]

    def moved(self, shape, placement):  # the toy measures everything at the origin
        return shape

    def parts(self, shape):
        return [], [] if isinstance(shape, Triangle) else shape, []

    def kind(self, shape):
        return ("Face", "Plane") if isinstance(shape, Triangle) else ("Edge", "Line")

    def center(self, shape):
        return 0, 0, 0

    def properties(self, shape):
        if isinstance(shape, Triangle):
            return {"refpoint": (0, 0, 0), "area": shape.size**2 / 2}
        return {"refpoint": (0, 0, 0)}

    def closest_points(self, shape1, shape2):
        return 0.0, (0, 0, 0), (0, 0, 0)

    def closest_directions(self, shape1, shape2):
        return None

register_show(KERNEL, ToyTessellation(), ToyShow(), api=1)
register_plugin(KERNEL, ToyBackend(), api=1)

show(Triangle(2), Segment((0, 0, 1), (2, 2, 1)), Frame((3, 0, 0)), names=["tri", "edge", "frame"], reset_camera="reset")
show([Triangle(1), Triangle(2)], names=["triangles"], reset_camera="reset")  # lists are the viewer's

no_rotation = (0, 0, 0, 1)
assembly = Assembly(
    [("a", Triangle(1), Placement((0, 0, 0), no_rotation)), ("b", Triangle(1), Placement((2, 0, 0), no_rotation))],
    location=Placement((0, 0, 5), no_rotation),
)
show(assembly, names=["assembly"], reset_camera="reset")  # /assembly/a and /assembly/b
```

## A plugin that measures: a kernel of points

ocp-viewer-core's `tests/toy_kernel.py`: a kernel whose bodies are lists of points, with no faces and no edges, so that every vertex is selectable, and the methods of [`BackendProtocol`](backend.md#backendprotocol). It is a test of the transport, not of geometry: `moved` applies only the translation of the placement. Its `serialize` is a plain function, which a real plugin's [`ShowProtocol`](show.md#serialize) would hold.

```python
import json
from math import sqrt

from ocp_viewer_core.backend import BackendProtocol, register_plugin

KERNEL = "toy"


def serialize(body):
    return json.dumps(body).encode("utf-8")


def deserialize(data):
    return [tuple(p) for p in json.loads(data.decode("utf-8"))]


def moved(shape, placement):
    (tx, ty, tz), _ = placement
    return [(x + tx, y + ty, z + tz) for x, y, z in shape]


def parts(shape):
    return [], [], shape


def kind(shape):
    return ("Compound", "Other") if isinstance(shape, list) else ("Vertex", "Point")


def center(shape):
    return shape[0] if isinstance(shape, list) else shape


def properties(shape):
    if isinstance(shape, list):
        return {"refpoint": shape[0]}
    return {"refpoint": shape, "point": shape}


def closest_points(shape1, shape2):
    p1 = shape1[0] if isinstance(shape1, list) else shape1
    p2 = shape2[0] if isinstance(shape2, list) else shape2
    return sqrt(sum((b - a) ** 2 for a, b in zip(p1, p2))), p1, p2


def closest_directions(shape1, shape2):
    return None


class ToyBackend(BackendProtocol):
    def deserialize(self, data):
        return deserialize(data)

    def moved(self, shape, placement):
        return moved(shape, placement)

    def parts(self, shape):
        return parts(shape)

    def kind(self, shape):
        return kind(shape)

    def center(self, shape):
        return center(shape)

    def properties(self, shape):
        return properties(shape)

    def closest_points(self, shape1, shape2):
        return closest_points(shape1, shape2)

    def closest_directions(self, shape1, shape2):
        return closest_directions(shape1, shape2)


BACKEND = ToyBackend()
register_plugin(KERNEL, BACKEND, api=1)
```

## What to test

### The drawing side

ocp-tessellate's `tests/test_external_objects.py` tests the pipeline's side with such a kernel: a body is meshed by the kernel with the viewer's `deviation` and `angular_tolerance` and shared under its cache id, edges and vertices come out as leaves with the right fields, every leaf has a tagged entry in the backend's model, a mixed scene keeps the OCCT objects measurable, colors, names, materials and states arrive as for OCCT, a location or axis has the points of an OCCT one at the converter's `helper_scale`, relative helper sizes see raw objects without `convert`, a forgotten method fails when the object is created, and a plugin is found through its entry point in a fresh process, with a broken plugin only warning, `compute_uvs` is true for an object with a material and a plugin may leave `uvs` out, an object nothing can draw raises when passed directly and is left out inside a list or dict, the methods are called by keyword, `api` is required and a version not served yet is refused, the tag `OCCT` is refused, what `convert`, `get_children` and `bounding_box` return is checked, a mesh with a wrong dtype, inconsistent counts or an unknown type code is refused, an alpha reaches an object without a color, a raw OCCT shape is never a plugin's, a kernel's leaf travels as base64 of its `serialize` bytes, and two plugins can use the same cache id. `tests/test_kernel_containers.py` tests `get_children` and `get_location` with a kernel that stores one assembly in two ways: the tree, the world positions, names, colors, children of any kernel and binding, the backend's entries and the bounding boxes.

For a real kernel, add on the plugin's side: its box against the OCCT box as a set of (vertex, normal) pairs, which must be identical, and the layout invariants of [`tessellate`](tessellation.md#tessellate) on curved bodies (normals aligned with vertices, indices in range, sums of `triangles_per_face` and `segments_per_edge` matching the buffers, winding agreeing with the normals, contiguous disjoint vertex blocks per face, every edge polyline within its deviation of the curve). For a `to_occt`, compare the converted bodies with the kernel's own: face counts, volumes and areas, validity (`BRepCheck_Analyzer`), and the lengths of converted edges.

### The measuring side

ocp-viewer-core's `tests/test_kernel_plugins.py` tests the core's side with the toy kernel: importing the plugin registers it, the entry point loads it in a process that never imported it, an unknown kernel names both ways to register, `api` is required and a version not served yet is refused, the tag `OCCT` is refused, the backend holds a plugin's body beside OCCT shapes under the ids of [The ids](backend.md#the-ids), the core runs `deserialize`, `moved` and `parts` and makes the panel from `kind` and `properties`, the distance tool asks for no `properties`, both tools answer for either kernel, a leaf that fails to load takes nothing else with it, a distance across kernels is one row, and the log lines of [Registration](backend.md#registration).

For a real kernel, add on the plugin's side:

- the order: every triangle of the tessellation's face block i lies on the face `parts` returns at index i, every segment of edge block i on edge i, and the vertices are `obj_vertices`. This is the test that catches a click reporting the wrong face
- the round trip: `BackendProtocol.deserialize` of `ShowProtocol.serialize` is x again for every kind of leaf, also for a body that was measured before it was serialized, `moved` moves it as OCCT's `tq_to_loc` moves an OCCT shape, and `deserialize` refuses bytes of another format version
- the measurements against an oracle: where the kernel's objects convert to OCCT, `ocp_viewer_core.measure` on the converted shapes against `measure.properties_response` and `measure.distance_response` on the plugin's answers, so they compare what the panel shows, paired by geometry (OCCT numbers sub-shapes in its own order); otherwise analytic values, such as the distance of two straight segments
- the whole way: the show pipeline's mapping, `encode_kernel_shapes`, the codec and JSON, `ViewerBackend.load_model`, and `handle_event` with ids taken from the mesh

ocp-viewer-smlibpy's `tests/test_backend.py` does all four.

## The reference plugin

[ocp-viewer-smlibpy](https://github.com/bernhard-42/ocp-viewer-smlibpy) implements all three protocols for SMLib, one class each: `SmlibTessellation` in `ocp_viewer_smlibpy/tessellation.py`, `SmlibShow` in `show.py`, `SmlibBackend` in `backend.py`, what the last two share in `_wire.py`, and its `tests/` the checks above, also that each process registers only its half, against the OCCT path and `ocp_viewer_core.measure` as oracles.
