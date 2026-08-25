# Object selection mode

![Object selection mode](./assets/selector.gif)

## Tool

![select-tool](./assets/select-tool.png) Click on faces, edges, or vertices to select their indices. Do not mix vertices, edges and faces and apply it to just one solid, compound, face or line.

## Topology Filter

For easier selection, there is a **topology filter**

![topo-filter](./assets/topo-filter.png)

There are keybinding shortcuts for the topology filters :

- "v" : vertices
- "e" : edges
- "f" : faces
- "s" : solid
- "n" : none

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

## Deselect

- In any of the tools pressing "escape" will delete all the selections
- "backspace" or "mouse button right click" will delete the last selection only.
