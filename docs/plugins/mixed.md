# Measuring objects of mixed kernels

The measurement backend measures the shapes of one kernel against each other: two smlib faces through the plugin's [`BackendProtocol`](backend.md#backendprotocol), two OCCT faces through ocp-viewer-core. A distance between a shape of the plugin's kernel and an OCCT shape, or a shape of another plugin, is not measured; the panel shows one row, `not measured: shapes of two kernels (smlib, OCCT)`. The kernels have no common geometry to measure on, except OCCT itself.

So mixed scenes are measured on the OCCT level. A plugin that can convert its objects to OCCT shapes offers `to_occt(obj, binding=None)`, and the user shows the converted object as an OCCT object, beside the other OCCT objects:

```python
from build123d import Box as BdBox, Location
from ocp_vscode import show
from smlibpy import Box, Cylinder, Pos
from ocp_viewer_smlibpy import to_occt

bracket = BdBox(10, 10, 10).moved(Location((40, 0, 0)))
plate = Box(20, 30, 10) - Pos(5, 5, 0) * Cylinder(4, 10)

show(bracket, to_occt(plate), names=["bracket", "plate"])  # every distance between the two is measured
```

The converted object is an OCCT shape from then on: OCCT tessellates it, numbers its faces and edges, and measures it. Names and colors are not part of the conversion; pass them to `show` as for any OCCT object. For objects of one kernel, the plain `show` stays the better way: the kernel's own tessellation and measurements, and no conversion that can fail.

## Writing `to_occt(obj, binding=None)`

Bodies go through an exchange format OCCT reads (the kernel's `.brep` writer and `BRepTools.Read_s`, or STEP), edges become exact lines or `Geom_BSplineCurve`s from the kernel's NURBS data, wires `BRepBuilderAPI_MakeWire`, points `BRepBuilderAPI_MakeVertex`, locations `TopLoc_Location`s from the 4x4 matrix (`gp_Trsf.SetValues`), planes `gp_Pln`s and axes `gp_Ax1`s; lists and dicts convert element-wise.

`binding` is `"OCP"` or `"nanocct"`, by default `ocp_tessellate._kernel.kernel()`. The collection classes differ between the bindings and OCP versions (`NCollection_Array1__gp_Pnt`, `TColgp_Array1OfPnt`, `OCP.collections.Array1_gp_Pnt`), and `BRepTools.Read_s` takes a text stream in nanocct and a bytes stream in OCP. For measuring the binding does not matter, since the backend rebuilds every OCCT shape from its BREP bytes in one binding; it matters when the user combines the converted shape with other OCCT shapes in their own code, e.g. `to_occt(obj, binding="OCP")` for CadQuery on OCP.

The conversion can fail for geometry the writer cannot express (smlibpy's `.brep` writer refuses offset surfaces of free-form faces). Raise a `ValueError` that names the viewer's `show` as the way to at least see the object.
