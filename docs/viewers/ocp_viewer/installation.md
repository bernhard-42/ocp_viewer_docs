# Installation

OCP Viewer is the standalone viewer: a local web server whose page is the [CAD Viewer](../../viewer.md), used from any editor or plain Python. All client-side features are available — the `show*` family, measurement mode, materials — except visual debugging, which is VS Code's.

```bash
pip install ocp_viewer
```

into the environment that also holds your CAD library (build123d or cadquery). Then start it:

```bash
python -m ocp_viewer
```

and open the printed URL in a browser — `http://127.0.0.1:3939` by default. The `ocp-viewer` console script does the same.

From Python, in the same environment:

```python
from build123d import *
from ocp_viewer import *

show(Box(1, 2, 3))
```

## Remote use over SSH

The server binds to `127.0.0.1`, which is unreachable from your local machine when it runs on a remote host. Rather than exposing it to the network with `--host`, forward the port over SSH:

```bash
ssh -L 3939:localhost:3939 user@host
```

then open `http://localhost:3939` in your local browser — the connection is tunnelled, and nothing is exposed beyond your machine.
