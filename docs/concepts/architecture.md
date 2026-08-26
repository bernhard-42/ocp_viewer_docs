# Architecture

Four viewers, one behavior — because almost everything is one shared implementation, and each viewer adds only what is genuinely its own.

![](../assets/viewer-ecosystem.png)

/// caption
Dependencies of the viewer ecosystem
///

## The pieces

- **[ocp-viewer-core](https://github.com/bernhard-42/ocp-viewer-core)** is the shared half, and it is two halves itself: a Python package on PyPI (the show pipeline, the configuration semantics, the measurement backend, the wire protocol) and a JavaScript package on npm (the viewer page that embeds the renderer, applies configs, and reports changes back). Both are published together under one version, so which version of the pair a viewer has is one question rather than two.
- **[three-cad-viewer](https://github.com/bernhard-42/three-cad-viewer)** is the renderer: the [CAD Viewer](../viewer.md) window itself, built on three.js. It knows nothing about CAD kernels — it draws tessellated meshes.
- **[ocp-tessellate](https://github.com/bernhard-42/ocp-tessellate)** turns OCP/build123d/CadQuery objects into those meshes — see [Mesh creation](tessellation.md).
- **The four viewer packages** — `ocp_vscode`, `ocp_viewer`, `jupyter_cadquery` (with `cad-viewer-widget`), `build123d_studio` — each supply exactly three things: a **transport** (how a message travels from Python to their embedded CAD Viewer — see [Communication](communication.md)), a **settings store** (see [Configuration layers](configuration.md)), and their own surface (a VS Code panel, a web page, a notebook sidecar, a desktop window).

This is why the docs can be shared: the behavior is not merely similar across viewers, it is the same code. A differing default or missing feature between viewers is treated as a defect, not a flavor.

## What happens when you call `show()`

The one narrative that ties the pieces together:

1. **Read** — the show asks the viewer two questions: its stored settings (`workspace_config`) and its live state (`status`), and merges them with `set_defaults` values and the call's own keywords into the effective configuration.
2. **Tessellate** — [ocp-tessellate](tessellation.md) converts the CAD objects into meshes, honoring `deviation` and `angular_tolerance`, reusing cached and repeated shapes.
3. **Send** — the model and its config block travel over the viewer's [transport](communication.md); the config keys are translated to the renderer's naming on the way out.
4. **Render** — the shared page hands the meshes to three-cad-viewer, applies the config, and positions the camera per `reset_camera`.
5. **Feed the backend** — the id-to-shape mapping of what was just drawn goes to the [measurement backend](backend.md), so later picks can be answered with exact geometry.
6. **Notify back** — from then on, everything you change in the viewer (a toggle, a slider, a selection) is reported back to Python as it happens, which is how `status()` answers without asking the browser and how the next show can preserve what you changed.

## The splash logo

Before the first `show`, every viewer displays the OCP splash logo — and it is a real model, shipped in two forms: as tessellated data for the renderer, and as measurable geometry for the backend, so even the splash can be measured. It carries its own display settings, which is why it must never be used to verify your configuration — the first real model is the honest test.
