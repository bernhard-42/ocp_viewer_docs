# The config system

All four viewers share the same three-level configuration, with the same precedence. Only where the first level is stored differs per viewer.

1. **Viewer settings** — the viewer's persistent configuration. In ocp_vscode these are the VS Code workspace settings, in ocp_viewer the `~/.ocpvscode_standalone` file and CLI flags, in Jupyter CadQuery and build123d Studio their own settings stores. See your viewer's documentation for where to edit them.
2. **`set_defaults(...)`** — per Python process. Overrides the viewer settings for every subsequent `show*` call in this process.
3. **Keywords on `show` / `show_object` / `show_objects` / `show_all`** — per call. Override both levels below, for this call only.

Note that not all parameters are available in the viewer settings, since some don't make sense globally (e.g. `helper_scale`, which depends on the size of the bounding box of the currently shown object).

There is a fourth participant that is not a level of its own: what you change in the running viewer — a toolbar toggle, a moved clipping slider — counts as viewer state and survives into the next `show` at the viewer-settings tier, so a toggle you clicked is not silently undone unless a `set_defaults` or a show keyword sets it explicitly. Inspect the effective result with `combined_config()` (see [Viewer state](#viewer-state) below).

A common setup:

```python
# %%
from build123d import *
import cadquery as cq

from ocp_viewer_core.viewer import *

set_defaults(reset_camera=Camera.CENTER, helper_scale=5)

# %%
...
```

- The first lines import build123d and CadQuery (omit what you are not interested in).
- The next line imports the complete viewer API, resolved to your viewer (see [Importing](importing.md)).
- `set_defaults` then sets `helper_scale` and `reset_camera` as defaults for every subsequent `show*` call.

## Defaults

- `set_defaults(**kwargs)`

    Persist values across subsequent `show*` calls in this Python process. Accepts the same viewer keywords as `show` (see [show.md](show.md)). The viewer's settings provide the starting values; `set_defaults` overrides them.

- `reset_defaults()`

    Re-apply the viewer's settings on top of the running viewer and clear the in-process defaults set via `set_defaults`.

- `get_default(key)` / `get_defaults()`

    Read a single default or the full merged default dict (viewer settings + `set_defaults`).

## Viewer state

- `status(debug=False)`

    Return the live state dict of the viewer (camera position, current tab, visibility states, …).

- `workspace_config()`

    Return the viewer's configuration as the viewer currently sees it. Raises `RuntimeError` if the viewer isn't reachable.

- `combined_config()`

    `workspace_config` merged with the live `status` and any `set_defaults` overrides — useful for "what will the next `show` actually use?" inspection.

The signatures show only the shared parameters; your viewer's variants additionally accept its addressing keyword — see "Addressing a viewer" in your viewer's chapter ([VS Code CAD Viewer](viewers/ocp_vscode/addressing.md), [OCP Viewer](viewers/ocp_viewer/addressing.md), [Jupyter CadQuery](viewers/jupyter_cadquery/addressing.md)).

