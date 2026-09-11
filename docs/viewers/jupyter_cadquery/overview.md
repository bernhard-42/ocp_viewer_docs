# Overview

Jupyter CadQuery is the viewer for JupyterLab: models appear in a sidecar panel beside the notebook, in a split window, or directly in a cell's output. It is a single Python package — the JupyterLab extension ships prebuilt inside its [cad-viewer-widget](https://github.com/bernhard-42/cad-viewer-widget) dependency, so there is no extension build step and no Node.js involved.

![](../../assets/jupyter-cadquery.png#only-light)
![](../../assets/jupyter-cadquery-dark.png#only-dark)

## Key features

- The complete [CAD Viewer](../../viewer.md) as a JupyterLab widget: navigation tree, clipping and zebra tools, [measurement and analysis tools](../../measure_select.md), [materials and Studio](../../pbr_studio.md) and [animation](../../animation.md) — it behaves exactly as in the other viewers, so all of that is documented once, in the shared chapters.
- [Sidecars, windows and cells](addressing.md): `open_viewer("CAD")` docks a viewer beside the notebook, `anchor="split-top"` and friends open it as a separate JupyterLab window, and without a sidecar the output lands in the cell itself. Sidecars and windows resize with their container, either filling it or keeping an aspect ratio.
- Several viewers at once, each addressed by its title rather than a port — including measuring in two viewers side by side.
- [Auto display](notebook.md#auto-display): a CadQuery or build123d object as the last expression of a cell is shown without a `show`.
- [Replay](replay.md) for CadQuery objects: step through the fluent-API calls that built an object and see each intermediate result.
- [Export](export.md) as a standalone HTML page, and notebooks converted with `nbconvert` keep their viewers.
- No sockets and no ports: the transport is Jupyter's own communication channel, so it works wherever JupyterLab works.
- Settings stored in `~/.jcq_config` — see [Workspace Config](workspace_config.md).
- Can be tried on binder without installing anything — see [Installation](installation.md#try-it-without-installing).

Outside JupyterLab — a plain Python process, an editor of your choice — the same `show` commands talk to [OCP Viewer](../ocp_viewer/overview.md) in a browser tab.

## In pictures

The viewer with a build123d hexapod, in a sidecar:

![Hexapod](https://raw.githubusercontent.com/bernhard-42/jupyter-cadquery/master/screenshots/hexapod.png)

[Measurement mode](../../measure.md): distances between objects, angles between edges and faces, and the properties (center, area, volume) of what is selected:

![Measurement](https://raw.githubusercontent.com/bernhard-42/jupyter-cadquery/master/screenshots/measure.gif)

The [animation system](../../animation.md) exploding an assembly around the origin:

![Exploded quadruped](https://raw.githubusercontent.com/bernhard-42/jupyter-cadquery/master/screenshots/explode.gif)

and a self-defined animation, the crawling hexapod:

![Animated hexapod](https://raw.githubusercontent.com/bernhard-42/jupyter-cadquery/master/screenshots/hexapod-crawling.gif)

The notebooks behind these pictures are in the repository's [examples folder](https://github.com/bernhard-42/jupyter-cadquery/tree/master/examples).

Continue with [Installation](installation.md).
