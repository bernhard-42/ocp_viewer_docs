# The config system

All four viewers share the same three-level configuration, with the same precedence. Only where the first level is stored differs per host.

1. **Host settings** — the viewer's persistent configuration. In ocp_vscode these are the VS Code workspace settings, in ocp_viewer the `~/.ocpvscode_standalone` file and CLI flags, in Jupyter CadQuery and build123d Studio their own settings stores. See your viewer's documentation for where to edit them.
2. **`set_defaults(...)`** — per Python process. Overrides the host settings for every subsequent `show*` call in this process.
3. **Keywords on `show` / `show_object` / `show_objects` / `show_all`** — per call. Override both levels below, for this call only.

Note that not all parameters are available in the host settings, since some don't make sense globally (e.g. `helper_scale`, which depends on the size of the bounding box of the currently shown object).

There is a fourth participant that is not a level of its own: what you change in the running viewer — a toolbar toggle, a moved clipping slider — counts as viewer state and survives into the next `show` at the host-settings tier, so a toggle you clicked is not silently undone unless a `set_defaults` or a show keyword sets it explicitly. Inspect the effective result with `combined_config()` (see [api.md](api.md)).

A common setup:

```python
# %%
from build123d import *
import cadquery as cq

from ocp_viewer import *           # if using ocp_viewer
# from ocp_vscode import *         # if using ocp_vscode
# from jupyter_cadquery import *   # if using jupyter_cadquery
# from build123d_studio import *   # if using build123d_studio

set_defaults(reset_camera=Camera.CENTER, helper_scale=5)

# %%
...
```

- The first block imports build123d and CadQuery (omit what you are not interested in).
- The second block imports all commands of your viewer's package.
- The third block sets `helper_scale` and `reset_camera` as defaults for every subsequent `show*` call.

## Keeping the camera orientation: `reset_camera`

Sometimes it is helpful to keep the orientation of an object across code changes. This is what `reset_camera` does:

- `reset_camera=Camera.CENTER` will keep position and rotation, but ignore panning. This means the new object will be repositioned to the center (most robust approach).
- `reset_camera=Camera.KEEP` will keep position, rotation and panning. However, panning can be problematic. When the next object to be shown is much larger or smaller and the object before was panned, it can happen that nothing is visible (the new object at the pan location is outside of the viewer frustum). The viewer checks whether the bounding box of an object is 2x smaller or larger than the one of the last shown object. If so, it falls back to `Camera.CENTER` and notifies via the host's log output.
- `reset_camera=Camera.RESET` will ensure that position, rotation and panning will be reset to the initial default.
- `reset_camera=Camera.ISO` / `Camera.TOP` / `Camera.BOTTOM` / `Camera.LEFT` / `Camera.RIGHT` / `Camera.FRONT` / `Camera.BACK` snaps to one of the axis-aligned camera presets — useful for reproducible screenshots.
