# Object selection mode

## Tool

![select-tool](./assets/select-tool.png) Click on faces, edges, or vertices to select their indices. Picking and filters are shared by all [analysis tools](measure_select.md).

When the selection holds one kind of one object, its indices are copied to the system clipboard as a comma-separated list, e.g. `6,16,7,17`, ready to be pasted into the `select_*` functions below. When it holds several objects or several kinds, the clipboard holds one comment line `# <object path> <kind>` per object and kind, each followed by its indices, so the indices of two objects never end up in one list:

```text
# /hexapod/top edges
3,5
# /hexapod/bottom faces
12
```

A selected solid gives its comment line only, `# <path> solid`, since it has no indices. Turning the tool off or pressing `ESC` leaves the clipboard as it is.

## Python functions

- `select_vertices(obj, indices)`

  Similar to `obj.vertices()` returns a `ShapeList` of vertices associated with the `indices` given as list, e.g. `[1,3,6]`

- `select_vertex(obj, index)`

  Shortcut for `select_vertices(obj, [index])[0]`

- `select_edges(obj, indices)`

  Similar to `obj.edges()` returns a `ShapeList` of edges associated with the `indices` given as list, e.g. `[1,3,6]`

- `select_edge(obj, index)`

  Shortcut for `select_edges(obj, [index])[0]`

- `select_faces(obj, indices)`

  Similar to `obj.faces()` returns a `ShapeList` of faces associated with the `indices` given as list, e.g. `[1,3,6]`

- `select_face(obj, index)`

  Shortcut for `select_faces(obj, [index])[0]`

!!! warning "Compatibility"

    The indices returned are stable across runs as long as no geometry change is introduced before `select_*`. However, Open Cascade does not guarantee the stability of the indices across major versions, e.g. from 7.x to 8.x.

## Example

**The object**

```python
from ocp_viewer_core.viewer import show, select_edges

from build123d import *

ccm = (Align.CENTER, Align.CENTER, Align.MIN)

b = Box(1, 1, 1)
b -= Cylinder(0.3, 0.5, align=ccm)
b -= Cylinder(0.4, 0.3, align=ccm)
b -= Box(0.3, 1.1, 0.5, align=ccm)
```

![Object selection mode](./assets/select-object.png#only-light){.center width=50%}
![Object selection mode](./assets/select-object-dark.png#only-dark){.center width=50%}

**Manual selection**

The manual selection leads to `31,35,28,15,44,19,45,36,12,26,22,39,38,27,13,11,33,43` being in the clipboard

![Object selection mode](./assets/select-selected-edges.png#only-light){.center width=50%}
![Object selection mode](./assets/select-selected-edges-dark.png#only-dark){.center width=50%}

**Filleting the manually selected edges**

```python
edges = select_edges(b, [
  31,35,28,15,44,19,45,36,12,26,22,39,38,27,13,11,33,43  # pasted from clipboard
])
b = fillet(edges, 0.05)
show(b)
```

![Object selection mode](./assets/select-filleted.png#only-light){.center width=50%}
![Object selection mode](./assets/select-filleted-dark.png#only-dark){.center width=50%}
