# The tessellation protocol

`TessellationProtocol` (`ocp_tessellate.plugin`) is what makes a kernel's shapes visible: it turns them into the meshes and line segments the viewer draws. OCCT implements the same methods in `ocp_tessellate.tessellator.OcctTessellation`, so the pipeline asks `tessellation_of(shape)` and treats every kernel alike. The shapes come from [`convert`](show.md#convert) of the show protocol: an `OcpObject` holds the kernel's shape, and goes the way an OCCT object goes.

A plugin subclasses the protocol, so a required method it forgot fails with a `TypeError` when its object is created. It names its kernel's types as the protocol's three parameters, the body, the edge and the vertex, so a type checker checks its methods against them; a plugin that names none works the same, unchecked. The pipeline calls the methods by keyword, so the parameter names below are part of the interface:

```python
from ocp_tessellate.plugin import TessellationProtocol

class SmlibTessellation(TessellationProtocol[Body, Edge, Vertex | Vector]):
    def tessellate(self, shape, deviation, quality, angular_tolerance, compute_edges=True,
                   compute_uvs=False, normalize_uvs=True, shape_id="", debug=False): ...
    def discretize_edges(self, edges, deflection=0.1, shape_id=""): ...
    def convert_vertices(self, vertices, shape_id=""): ...
    def bounding_box(self, shape): ...
    def cache_id(self, shape): ...  # optional
```

| method | required | what it gives |
|---|---|---|
| [`tessellate`](#tessellate) | yes | the mesh of a solid, shell, sheet or face |
| [`discretize_edges`](#discretize_edges) | yes | the line segments of edges |
| [`convert_vertices`](#convert_vertices) | yes | the coordinates of vertices |
| [`bounding_box`](#bounding_box) | yes | the box of a shape or a list of edges or vertices |
| [`cache_id`](#cache_id) | no | a key that shares and caches the mesh; without it the body is meshed at every show |

## `tessellate`

`tessellate(shape, deviation, quality, angular_tolerance, compute_edges, compute_uvs, normalize_uvs, shape_id, debug) -> Tessellation`

The triangulation of a solid, shell, sheet or face, in the layout the viewer wants. The pipeline calls it through its one cache, with the values the host resolved. `shape_id` and `debug` may be ignored, and `deviation` too unless the kernel computes UVs:

| argument | meaning |
|---|---|
| `quality` | the chord deviation to mesh to, in model units: `compute_quality(box, deviation)` of `ocp_tessellate.tessellator` on the plugin's [`bounding_box`](#bounding_box), the mean size of the box divided by 100, times the viewer's `deviation` (default 0.1) |
| `angular_tolerance` | the largest angle between neighbouring facet normals, in radians (default 0.2) |
| `compute_edges` | whether the edges are wanted: `False` for `render_edges=False`. Then `edges` is empty, and `segments_per_edge` holds zeros |
| `compute_uvs` | whether UVs are wanted, see [UVs](#uvs) |
| `normalize_uvs` | how they are scaled, see [UVs](#uvs) |

The `Tessellation` it returns is a dict of flat numpy arrays (`ocp_tessellate.types.Tessellation`), laid out for three.js:

| key | dtype | content |
|---|---|---|
| `vertices` | float32, 3 V | the points, face by face: every face owns a contiguous block of vertices, and a point on the border between two faces appears once per face |
| `normals` | float32, 3 V | one normal per vertex, the surface normal of the vertex's face at that point; a point where the normals of one face differ (the apex of a cone) is split into one vertex per normal |
| `triangles` | int32, 3 T | vertex indices, counter-clockwise seen from outside, the triangles of a face contiguous |
| `triangles_per_face` | int32, F | triangles of each face, in face order |
| `face_types` | int32, F | `GeomAbs_SurfaceType` values, by name in `ocp_tessellate.plugin.SURFACE_TYPES`: `Plane` 0, `Cylinder` 1, `Cone` 2, `Sphere` 3, `Torus` 4, `BezierSurface` 5, `BSplineSurface` 6, `SurfaceOfRevolution` 7, `SurfaceOfExtrusion` 8, `OffsetSurface` 9, `OtherSurface` 10 |
| `edges` | float32, 6 S | the segments of the edges as point pairs, edge by edge |
| `segments_per_edge` | int32, E | segments of each edge, in edge order; 0 for an edge without segments |
| `edge_types` | int32, E | `GeomAbs_CurveType` values, by name in `ocp_tessellate.plugin.CURVE_TYPES`: `Line` 0, `Circle` 1, `Ellipse` 2, `Hyperbola` 3, `Parabola` 4, `BezierCurve` 5, `BSplineCurve` 6, `OffsetCurve` 7, `OtherCurve` 8 |
| `obj_vertices` | float32, 3 O | the body's vertices |
| `uvs` | float32, 2 V | optional: one (u, v) per vertex, see [UVs](#uvs) |

The pipeline checks a plugin's result before it goes on, since it goes to the viewer without a further look: every key there, the dtypes of the table, counts that agree with the buffers, and type codes that are in `SURFACE_TYPES` and `CURVE_TYPES`, since the viewer looks up their names (`validate_tessellation`, `validate_edges` and `validate_vertices` in `ocp_tessellate.tessellator`, which raise a `ValueError` that names the array). So a dtype is not a suggestion: numpy's default `float64` is refused. `validate_tessellation` is in `ocp_tessellate.plugin` too, for a plugin's tests.

Faces, edges and vertices are numbered in the order of these arrays: the viewer names them `faces_i`, `edges_i`, `vertices_i` for picking, so a face without triangles still takes an entry (0 in `triangles_per_face`, its type in `face_types`), and an edge without segments one in `segments_per_edge`.

Edges should be sampled so that every segment's chord lies within a tenth of `quality` of the curve, which is what the OCCT path does (`deflection = quality / 10`). A straight edge is one segment.

Welded meshes, where faces share points and normals belong to triangle corners, have to be unwelded: sort the triangles by face, and within a face give one vertex to each distinct (point, normal) pair.

### UVs

The pipeline sets `compute_uvs` only for an object with a material, since UVs map its textures. A plugin that cannot compute them leaves `uvs` out: the object is shown as usual and its mesh has no UVs. So a kernel can gain UVs later by changing its plugin alone, with nothing to change in the protocol or the pipeline.

When a plugin returns `uvs`, they are in the order of `vertices`, and `normalize_uvs` says how they are scaled:

| `normalize_uvs` | (u, v) of a vertex |
|---|---|
| `True`, the default | `((u - u_min) * scale_u / size, (v - v_min) * scale_v / size)`: `u_min` and `v_min` are the lower bounds of the face's parameters, `scale_u` and `scale_v` the physical length per unit of the parameter at the centre of the face, and `size` is `quality * 100 / deviation`, the mean size of the box. One UV unit is that size, so a texture keeps its aspect ratio and tile size across faces |
| `False` | the face's raw parameters as `(u, 1 - v)`, the origin at the top left as in glTF |

A vertex on the seam of a closed surface has two parameter values. It needs one vertex per side, or the triangles across the seam stretch over the whole texture.

### Example: the unit box

`Box(1, 1, 1)` centred at the origin, as smlibpy tessellates it (identical to the OCCT path as a set of vertex and normal pairs). Six faces with four vertices each, so 24 vertices for 8 corner points: every corner appears once per face that meets there, with that face's normal.

```python
{
    # 24 vertices, a block of 4 per face; face f owns vertices 4f .. 4f+3
    "vertices": np.array([
        [-0.5, -0.5, -0.5], [-0.5, 0.5, -0.5], [0.5, 0.5, -0.5], [0.5, -0.5, -0.5],   # face 0, normal [0, 0, -1]
        [-0.5, -0.5, 0.5], [0.5, -0.5, 0.5], [-0.5, 0.5, 0.5], [0.5, 0.5, 0.5],   # face 1, normal [0, 0, 1]
        [-0.5, -0.5, -0.5], [-0.5, 0.5, -0.5], [-0.5, -0.5, 0.5], [-0.5, 0.5, 0.5],   # face 2, normal [-1, 0, 0]
        [0.5, 0.5, -0.5], [0.5, -0.5, -0.5], [0.5, -0.5, 0.5], [0.5, 0.5, 0.5],   # face 3, normal [1, 0, 0]
        [-0.5, -0.5, -0.5], [0.5, -0.5, -0.5], [-0.5, -0.5, 0.5], [0.5, -0.5, 0.5],   # face 4, normal [0, -1, 0]
        [-0.5, 0.5, -0.5], [0.5, 0.5, -0.5], [-0.5, 0.5, 0.5], [0.5, 0.5, 0.5],   # face 5, normal [0, 1, 0]
    ], dtype=np.float32).ravel(),
    # one normal per vertex: the face normal, 4 times per face
    "normals": np.array([
        *[[0, 0, -1]] * 4,   # face 0
        *[[0, 0, 1]] * 4,   # face 1
        *[[-1, 0, 0]] * 4,   # face 2
        *[[1, 0, 0]] * 4,   # face 3
        *[[0, -1, 0]] * 4,   # face 4
        *[[0, 1, 0]] * 4,   # face 5
    ], dtype=np.float32).ravel(),
    # 12 triangles, 2 per face, indices into the face's own block
    "triangles": np.array([
        [0, 2, 3], [2, 0, 1],   # face 0
        [4, 7, 6], [7, 4, 5],   # face 1
        [8, 11, 9], [11, 8, 10],   # face 2
        [13, 15, 14], [15, 13, 12],   # face 3
        [16, 19, 18], [19, 16, 17],   # face 4
        [20, 23, 21], [23, 20, 22],   # face 5
    ], dtype=np.int32).ravel(),
    "triangles_per_face": np.array([2, 2, 2, 2, 2, 2], dtype=np.int32),
    "face_types": np.array([0, 0, 0, 0, 0, 0], dtype=np.int32),   # all planes
    # 12 edges, one segment each: the two end points of every edge
    "edges": np.array([
        [0.5, -0.5, 0.5], [0.5, 0.5, 0.5], [-0.5, 0.5, 0.5], [0.5, 0.5, 0.5],
        [-0.5, -0.5, -0.5], [-0.5, 0.5, -0.5], [-0.5, -0.5, 0.5], [-0.5, 0.5, 0.5],
        [0.5, -0.5, -0.5], [0.5, 0.5, -0.5], [0.5, 0.5, -0.5], [0.5, 0.5, 0.5],
        [-0.5, -0.5, -0.5], [-0.5, -0.5, 0.5], [-0.5, -0.5, -0.5], [0.5, -0.5, -0.5],
        [0.5, -0.5, -0.5], [0.5, -0.5, 0.5], [-0.5, -0.5, 0.5], [0.5, -0.5, 0.5],
        [-0.5, 0.5, -0.5], [-0.5, 0.5, 0.5], [-0.5, 0.5, -0.5], [0.5, 0.5, -0.5],
    ], dtype=np.float32).ravel(),
    "segments_per_edge": np.array([1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1], dtype=np.int32),
    "edge_types": np.array([0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0], dtype=np.int32),   # all lines
    # the 8 corner points
    "obj_vertices": np.array([
        [0.5, 0.5, 0.5], [-0.5, -0.5, 0.5], [-0.5, -0.5, -0.5], [0.5, -0.5, -0.5],
        [0.5, -0.5, 0.5], [-0.5, 0.5, 0.5], [-0.5, 0.5, -0.5], [0.5, 0.5, -0.5],
    ], dtype=np.float32).ravel(),
}
```

## `discretize_edges`

`discretize_edges(edges, deflection, shape_id) -> DiscretizedEdges`

The segments of edges shown on their own or drawn as a helper, as a list. `DiscretizedEdges` is a dict with `edges`, `segments_per_edge`, `edge_types` and `obj_vertices` as in [`tessellate`](#tessellate), `obj_vertices` the end points. The chord deviation is `deflection`, which the pipeline computes as it does for OCCT: `edge_accuracy` when the viewer has one, else a tenth of `compute_quality(bounding_box(edges), deviation)`. A straight edge is one segment.

### Example: a rectangle with an arc

A wire of three lines and an arc, the 2 x 1 rectangle whose top side is the arc through (1, 1.5, 0), radius 1.25:

```python
wire = Wire([Line((0, 0, 0), (2, 0, 0)), Line((2, 0, 0), (2, 1, 0)),
             ThreePointArc((2, 1, 0), (1, 1.5, 0), (0, 1, 0)), Line((0, 1, 0), (0, 0, 0))])
discretize_edges(wire.edges(), deflection=compute_quality(bounding_box(wire.edges()), 0.1) / 10)
```

The box is 2 x 1.5, so with a `deviation` of 0.1 `quality` is about 0.0012 and the chord deviation a tenth of that. Each line is one segment; the arc gets 76 segments. Four edges, so `segments_per_edge` and `edge_types` have four entries; `obj_vertices` holds the four corners once each.

```python
{
    # 79 segments, point pairs (start, end), edge by edge
    "edges": np.array([
        [0, 0, 0], [2, 0, 0],   # edge 0, a line: one segment
        [2, 0, 0], [2, 1, 0],   # edge 1, a line: one segment
        [2, 1, 0], [1.9814, 1.0242, 0],   # edge 2, the arc: 76 segments
        [1.9814, 1.0242, 0], [1.9622, 1.0479, 0],
        ...,
        [0.0186, 1.0242, 0], [0, 1, 0],
        [0, 1, 0], [0, 0, 0],   # edge 3, a line: one segment
    ], dtype=np.float32).ravel(),
    "segments_per_edge": np.array([1, 1, 76, 1], dtype=np.int32),
    "edge_types": np.array([0, 0, 1, 0], dtype=np.int32),   # line, line, circle, line
    "obj_vertices": np.array([[0, 0, 0], [2, 0, 0], [2, 1, 0], [0, 1, 0]], dtype=np.float32).ravel(),
}
```

## `convert_vertices`

`convert_vertices(vertices, shape_id) -> ConvertedVertices`

The counterpart of `discretize_edges` for vertices: `{"obj_vertices": coordinates}`, a flat float32 array of the points.

## `bounding_box`

`bounding_box(shape) -> BoundingBox`

The box of a shape, or a list of edges or vertices, as an `ocp_tessellate.ocp_utils.BoundingBox`, built from the kernel's extremes: `BoundingBox({"xmin": ..., "xmax": ..., "ymin": ..., "ymax": ..., "zmin": ..., "zmax": ...})`. It gives `quality` (`compute_quality`), the camera and relative `helper_scale` values. A flat body has a zero extent, which is fine, but a body always has a box: `None` for one raises a `TypeError` that names the plugin's class. A location, plane or axis has no extent: raise or return `None`, and it adds nothing to the box that sizes relative helpers. The pipeline asks it on the raw object, as it measures an OCCT object, without calling `convert`.

## `cache_id`

`cache_id(shape) -> str | None`, optional

A string that identifies the body's geometry, not the Python object, nor its color or name: equal for a body built twice, different for a moved copy. The mesh goes into the cache of the OCCT tessellator (`ocp_tessellate.tessellator.cache`, keyed as `make_key` keys OCCT shapes: `cache_id`, `deviation`, `angular_tolerance`, `compute_edges`, `compute_faces`, `compute_uvs`, `normalize_uvs`), and the same id in one show shares one instance, as OCCT's shapes do. A body built twice with the same `cache_id` is meshed once; the progress line shows `+` for a mesh and `c` for a hit. The default, `None`, means the body is meshed at every show.

The pipeline keeps the ids of two plugins apart by their kernel tags, so a plugin need not make its ids unique beyond its own kernel. The bounding box of an instance is cached under the same id, so what the id leaves out does not change the box either.

The mesh of one instance is drawn once per placement, so a body drawn in many places shares one mesh only if the kernel keeps the placement apart from the geometry: `convert` gives the object's geometry, and `get_location` or `get_children` its placement. In a kernel whose moved copies hold moved geometry, each copy is a body of its own, with an id of its own.

```python
def cache_id(self, shape: _Body) -> str:
    return hashlib.sha256(to_smb(shape, ascii=False)).hexdigest()  # smlib's binary native file: 3 ms for 180 faces
```
