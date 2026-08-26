# Mesh creation

CAD kernels describe geometry exactly — a cylinder is a mathematical cylinder (a _BRep_, boundary representation). Screens draw triangles. **Tessellation** is the conversion, done by [ocp-tessellate](https://github.com/bernhard-42/ocp-tessellate) on every `show`, and two consequences of it explain most of what you see:

- what the CAD Viewer renders is an _approximation_, controlled by two knobs
- what the [measurement backend](backend.md) answers from is the _exact_ BRep — which is why measured numbers can differ from what the mesh suggests

## The two knobs

- **`deviation`** (default 0.1) — how far a mesh face may sit from the true surface, relative to the object size. Smaller: rounder circles, more triangles, slower.
- **`angular_tolerance`** (default 0.2 rad) — the maximum angle a tessellated edge segment may span. Smaller: smoother curved edges.

Both are show keywords and viewer settings; edges are additionally discretized for display with `edge_accuracy`.

## What keeps it fast

- **Caching** — a shape already tessellated at the same quality is not tessellated again; re-showing during iteration reuses the previous run's work.
- **Instances** — identical shapes (the same screw placed forty times) are recognized by hash and tessellated once; the wire carries one buffer and forty locations. On the viewer side the instances are then resolved back into individual meshes before rendering — three.js's instanced rendering has severe problems with transparency, so the savings are in tessellation time and transfer size, not on the GPU.

The `progress` string a show prints is exactly this machinery reporting per object: `-` reused as a reference (instance), `+` tessellated in Python, `*` tessellated natively, `c` served from cache.

## From mesh to screen

The tessellation — vertex, triangle, normal and edge buffers plus the object tree — travels over the viewer's [transport](communication.md) and is handed to three-cad-viewer, which builds three.js geometry directly on those buffers. The navigation tree you click is the same tree the tessellator built; the shape ids the [analysis tools](../measure_select.md) report are indices into it, which is why they stay valid exactly as long as the model is unchanged.
