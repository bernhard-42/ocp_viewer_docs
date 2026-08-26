# Measure mode

![Measurement mode](./assets/measure.gif#only-light)
![Measurement mode](./assets/measure-dark.gif#only-dark)

There are two measurement tools; the picking mechanics and the mesh-based vs CAD-exact distinction are shared by all [analysis tools](measure_select.md).

![properties-tool](./assets/properties-tool.png) **Properties**: Get the properties of the object selected. For circles and ellipses the diameter is also reported.

![measure-tool](./assets/measure-tool.png) **Measurement**: Get the distance of the two objects selected and the angle between them or their normals.

## Distance

Pick two shapes; an arrow with a result panel appears. Holding `shift` while picking the **second** shape measures center-to-center instead of the minimum distance — the panel says `(min)` or `(center)` accordingly. The panel shows:

- **distance**, and its **X | Y | Z** components
- **point 1** and **point 2**, the exact points measured between
- when both picks are edges or faces, the **angle** between them, with a reference row per pick saying what was compared: `line` or `tangent at P1/P2` for edges, `face normal` or `surface normal at P1/P2` for faces, plus the direction/normal vectors. For an edge against a face the angle is given relative to the surface.

![distance](assets/viewer-distance.png#only-light){ .center width="48%" }
![distance-dark](assets/viewer-distance-dark.png#only-dark){ .center width="48%" }

/// caption
Distance measurement using a Python backend
///

## Properties

Pick one shape; the panel reports what the geometry kind warrants:

- **Vertex** — its `xyz` position
- **Edge** — by curve type: a line's start/middle/end; a circle's center and `radius / diam`; an ellipse's center and major/minor `radius / diam`; hyperbola/parabola vertex and endpoints; plus always the exact **length** and, where computable, the tangent angles to the XY plane at both ends
- **Face** — by surface type: a plane's center; a cylinder's center and radius; a cone's base radius and half angle; a sphere's radius; a torus's major/minor radius; a revolution's axis; plus always the exact **area** and the mid-face normal's angle to XY
- **Solid / Compound** — the exact **volume**
- everything except vertices also gets its **bounding box** (min, center, max, size)

![properties](assets/viewer-properties.png#only-light){ .center width="48%" }
![properties-dark](assets/viewer-properties-dark.png#only-dark){ .center width="48%" }

/// caption
Properties provided by a Python backend
///
