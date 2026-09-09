# Installation

OCP Viewer is the standalone viewer: a local web server whose page is the [CAD Viewer](../../viewer.md), used from any editor or plain Python. All client-side features are available — the `show*` family, measurement mode, materials — except visual debugging, which is VS Code's.

```bash
pip install ocp_viewer
```

into the environment that also holds your CAD library (build123d or cadquery). Then start it:

```bash
python -m ocp_viewer
```

and open the printed URL in a browser — `http://127.0.0.1:3939` by default. The `ocp-viewer` console script does the same, and takes the same flags. Stop the server with Ctrl-C; a page that is still open keeps trying to reconnect until it is closed or the server is back.

From Python, in the same environment:

```python
from build123d import *
from ocp_viewer import *

show(Box(1, 2, 3))
```

## Installing from a checkout

For local use straight from a clone — `uv add path/to/ocp-viewer`, `uv pip install path/to/ocp-viewer` or `pip install path/to/ocp-viewer` — run `make assets` in the checkout first. The page's JavaScript and stylesheet are copied in from npm and are not in git, so a package built from a checkout that never ran it ships without them; a server started from such a package says so and names the files. `make dist` runs it for you, and a released wheel from PyPI carries them.

## Remote use over SSH

The server binds to `127.0.0.1`, which is unreachable from your local machine when it runs on a remote host. Rather than exposing it to the network with `--host`, forward the port over SSH:

```bash
ssh -L 3939:localhost:3939 user@host
```

then open `http://localhost:3939` in your local browser — the connection is tunnelled, and nothing is exposed beyond your machine.
