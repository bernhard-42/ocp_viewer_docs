# Overview

Jupyter CadQuery is the viewer for JupyterLab: models appear in a sidecar panel beside the notebook, or directly in a cell's output. It is a single Python package — the JupyterLab extension ships prebuilt inside its [cad-viewer-widget](https://github.com/bernhard-42/cad-viewer-widget) dependency, so there is no extension build step and no Node.js involved.

![](../../assets/jupyter-cadquery.png#only-light)
![](../../assets/jupyter-cadquery-dark.png#only-dark)

## Key features

- The complete [CAD Viewer](../../viewer.md) as a JupyterLab widget: navigation tree, clipping and zebra tools, measurement and analysis tools, [materials](../../pbr_studio.md) and [animation](../../animation.md).
- Sidecars and cells: `open_viewer("CAD")` docks a viewer beside the notebook, and without one the output lands in the cell itself.
- Several viewers at once, each addressed by its title rather than a port — including measuring in two viewers side by side — see [Addressing a viewer](addressing.md).
- No sockets and no ports: the transport is Jupyter's own communication channel, so it works wherever JupyterLab works.
- Settings stored in `~/.jcq_config` — see [Workspace Config](workspace_config.md).
- Can be tried on binder without installing anything — see [Installation](installation.md#try-it-without-installing).

Continue with [Installation](installation.md).
