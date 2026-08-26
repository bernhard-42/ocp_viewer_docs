# Installation

Jupyter CadQuery is the viewer for JupyterLab: a JupyterLab extension plus a Jupyter Server extension, showing CadQuery, build123d and OCP objects in notebook cells and sidecars.

```bash
pip install jupyter-cadquery
```

into the environment that also holds your CAD library. It requires JupyterLab 4 (`jupyterlab>=4.6.2,<5`); the JupyterLab extension is prebuilt and ships inside the [cad-viewer-widget](https://github.com/bernhard-42/cad-viewer-widget) dependency, so no `jupyter labextension` step and no Node.js are needed.

Then start JupyterLab as usual:

```bash
jupyter lab
```

and in a notebook:

```python
from build123d import *
from jupyter_cadquery import *

open_viewer("CAD")
show(Box(1, 2, 3))
```

`open_viewer` puts the viewer into a sidecar panel next to the notebook; without it, `show` opens the default sidecar on first use — see [Addressing a viewer](addressing.md) for titles, anchors and multiple sidecars.

## Try it without installing

The [binder link in the repository](https://github.com/bernhard-42/jupyter-cadquery#readme) starts a live JupyterLab with everything installed. Due to binder's security restrictions, the measurement feature does not work there.
