# CAD kernel plugins

The viewers show and measure the objects of OCCT, the CAD kernel underneath build123d and CadQuery. A **kernel plugin** adds another kernel: its bodies are tessellated by that kernel, go through the same show pipeline as OCCT objects, and are measured by the [analysis tools](../measure_select.md) from that kernel's exact geometry. Names, colors, the viewer's settings, every `show` keyword and `set_defaults` work for them as for build123d objects.

The first plugin is [ocp-viewer-smlibpy](https://github.com/bernhard-42/ocp-viewer-smlibpy) for SMLib through [smlibpy](https://pypi.org/project/smlibpy/). It is the reference implementation of everything in this chapter.

## Using a plugin

Install the plugin, and the viewer's own [show](../show.md) and [show_all](../show_all.md) take the kernel's objects like any other, also beside build123d, CadQuery or OCCT objects and at a debug stop. Your script imports only the kernel; the viewer imports every installed plugin at the first show. With ocp-viewer-smlibpy:

```python
from smlibpy import Box, Cylinder, Pos
from ocp_vscode import show, show_all

plate = Box(80, 60, 10) - Pos(20, 0, 0) * Cylinder(8, 10)

show(plate, plate.edges(), names=["plate", "edges"])
show_all()   # every shape in the current scope, under its variable name
```

- `show` and `show_all` take parts, sheets, sketches, faces, edges, wires, vertices, vectors, locations, planes and axes, and lists or dicts of them, with `names`, `colors`, `alphas` and every keyword of [show](../show.md).
- Everything shown is measured by the Distance and Properties tools like OCCT objects, bodies, edges, vertices, locations, planes and axes: radii and centers of circles, cylinders, cones, spheres and tori, areas, lengths, volumes, distances and angles, all from smlib's geometry.
- Objects of several kernels can be in one scene: `show(plate, build123d_box)`. Each kernel's shapes are measured against each other, but a distance between an smlib shape and an OCCT shape is not; the panel says so.
- To measure across kernels, convert to OCCT: `show(build123d_box, to_occt(plate))`, with `to_occt` from `ocp_viewer_smlibpy`, shows both as OCCT shapes, and every distance between them is measured, see [Measuring objects of mixed kernels](mixed.md).

The plugin must be installed where the viewer's [measurement backend](../concepts/backend.md) runs, not only where your script runs: for VS Code in the interpreter the extension starts the backend with, for OCP Viewer in the environment `ocp_viewer` runs in, for Jupyter CadQuery in the Jupyter server's environment, and for build123d Studio in its own environment, through [Additional packages](../viewers/build123d_studio/packages.md#additional-packages) (installing restarts the measurement backend, which then loads the plugin). If objects of the kernel are not drawn or not measured, see [When it does not work](entry_points.md#when-it-does-not-work). Install it from PyPI:

```bash
pip install ocp-viewer-smlibpy
```

## Writing a plugin

A plugin is for a kernel with bodies, faces, edges and vertices of exact geometry: the viewer's tools pick and measure those. Kernels that only hold meshes are out of scope.

A plugin is a Python package with one class per protocol, two towards ocp-tessellate, which draws, and one towards ocp-viewer-core, which measures. Each is a Python protocol to subclass, so a required method it forgot fails when its object is created:

| protocol | what it does | required methods | optional methods | from |
|---|---|---|---|---|
| [`TessellationProtocol`](tessellation.md) | the kernel's shapes as meshes, in the layout the viewer wants | `tessellate`, `discretize_edges`, `convert_vertices`, `bounding_box` | `cache_id` | `ocp_tessellate.plugin` |
| [`ShowProtocol`](show.md) | makes the viewer's `show` and `show_all` take the kernel's objects, and writes their bytes | `is_object`, `convert`, `line`, `serialize` | `get_children`, `get_location` | `ocp_tessellate.plugin` |
| [`BackendProtocol`](backend.md) | the kernel's shapes from their bytes, and what the kernel knows about them | `deserialize`, `moved`, `parts`, `kind`, `center`, `properties`, `closest_points`, `closest_directions` | | `ocp_viewer_core.backend` |

All three are required. The pipeline calls their methods by keyword, so the parameter names are part of the interface. The package then does two more things, without which the protocols are never called: each process's half registers when its module is imported, stating the version of the interface it was written against (`api=1`), and it declares two entry points in its `pyproject.toml`, keyed by its kernel tag, through which the viewer imports the drawing half in the user's process and the measuring half in the backend's. [Registration and entry points](entry_points.md) describes them:

```toml
[project.entry-points."ocp_viewer.kernels.show"]
smlib = "ocp_viewer_smlibpy.show"

[project.entry-points."ocp_viewer.kernels.backend"]
smlib = "ocp_viewer_smlibpy.backend"
```

Optional is a conversion to OCCT shapes, `to_occt`, which measures the plugin's objects against OCCT objects, see [Measuring objects of mixed kernels](mixed.md). [Toy plugins and testing](examples.md) has two complete minimal plugins, one for drawing and one for measuring, and what to test for a real kernel.

What the pipeline does for a plugin: names and unique names, colors and the default color of each kind, the viewer's config and every `show` keyword, the resolution of `deviation`, `angular_tolerance` and `helper_scale` from keywords and `set_defaults`, the tree of groups and parts, the wire format, sending, caching of meshes, the panel's rows and strings, and mixing with build123d, CadQuery and OCCT objects in one scene. What a plugin does: tessellate, and tell what its kernel knows about a shape.

A plugin needs ocp-tessellate 3.7 and ocp-viewer-core 1.2 or later. ocp-tessellate still needs an OCCT binding, OCP or nanocct, installed beside the plugin's kernel.
