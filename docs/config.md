# The config system

All four viewers share the same three-level configuration, with the same precedence. Only where the first level is stored differs per host.

1. **Host settings** — the viewer's persistent configuration. In ocp_vscode these are the VS Code workspace settings, in ocp_viewer the `~/.ocpvscode_standalone` file and CLI flags, in Jupyter CadQuery and build123d Studio their own settings stores. See your viewer's documentation for where to edit them.
2. **`set_defaults(...)`** — per Python process. Overrides the host settings for every subsequent `show*` call in this process.
3. **Keywords on `show` / `show_object` / `show_objects` / `show_all`** — per call. Override both levels below, for this call only.

Note that not all parameters are available in the host settings, since some don't make sense globally (e.g. `helper_scale`, which depends on the size of the bounding box of the currently shown object).

There is a fourth participant that is not a level of its own: what you change in the running viewer — a toolbar toggle, a moved clipping slider — counts as viewer state and survives into the next `show` at the host-settings tier, so a toggle you clicked is not silently undone unless a `set_defaults` or a show keyword sets it explicitly. Inspect the effective result with `combined_config()` (see [Viewer state](#viewer-state) below).

A common setup:

=== "ocp_viewer"

    ```python
    # %%
    from build123d import *
    import cadquery as cq

    from ocp_viewer import *

    set_defaults(reset_camera=Camera.CENTER, helper_scale=5)

    # %%
    ...
    ```

=== "ocp_vscode"

    ```python
    # %%
    from build123d import *
    import cadquery as cq

    from ocp_vscode import *

    set_defaults(reset_camera=Camera.CENTER, helper_scale=5)

    # %%
    ...
    ```

=== "jupyter_cadquery"

    ```python
    # %%
    from build123d import *
    import cadquery as cq

    from jupyter_cadquery import *

    set_defaults(reset_camera=Camera.CENTER, helper_scale=5)

    # %%
    ...
    ```

=== "build123d_studio"

    ```python
    # %%
    from build123d import *
    import cadquery as cq

    from build123d_studio import *

    set_defaults(reset_camera=Camera.CENTER, helper_scale=5)

    # %%
    ...
    ```

- The first lines import build123d and CadQuery (omit what you are not interested in).
- The next line imports all commands of your viewer's package (pick your viewer's tab).
- `set_defaults` then sets `helper_scale` and `reset_camera` as defaults for every subsequent `show*` call.

## Defaults

- `set_defaults(**kwargs)`

    Persist values across subsequent `show*` calls in this Python process. Accepts the same viewer keywords as `show` (see [show.md](show.md)). The host's settings provide the starting values; `set_defaults` overrides them.

- `reset_defaults()`

    Re-apply the host's settings on top of the running viewer and clear the in-process defaults set via `set_defaults`.

- `get_default(key)` / `get_defaults()`

    Read a single default or the full merged default dict (host settings + `set_defaults`).

## Viewer state

- `status(debug=False)`

    Return the live state dict of the viewer (camera position, current tab, visibility states, …).

- `workspace_config()`

    Return the host's configuration as the viewer currently sees it. Raises `RuntimeError` if the viewer isn't reachable.

- `combined_config()`

    `workspace_config` merged with the live `status` and any `set_defaults` overrides — useful for "what will the next `show` actually use?" inspection.

The signatures show only the shared parameters; your viewer's variants additionally accept its addressing keyword — see "Addressing a viewer" in your viewer's chapter ([VS Code CAD Viewer](hosts/ocp_vscode/addressing.md), [OCP Viewer](hosts/ocp_viewer/addressing.md), [Jupyter CadQuery](hosts/jupyter_cadquery/addressing.md)).

## Keeping the camera orientation: `reset_camera`

Sometimes it is helpful to keep the orientation of an object across code changes. This is what `reset_camera` does:

- `reset_camera=Camera.CENTER` will keep position and rotation, but ignore panning. This means the new object will be repositioned to the center (most robust approach).
- `reset_camera=Camera.KEEP` will keep position, rotation and panning. However, panning can be problematic. When the next object to be shown is much larger or smaller and the object before was panned, it can happen that nothing is visible (the new object at the pan location is outside of the viewer frustum). The viewer checks whether the bounding box of an object is 2x smaller or larger than the one of the last shown object. If so, it falls back to `Camera.CENTER` and notifies via the host's log output.
- `reset_camera=Camera.RESET` will ensure that position, rotation and panning will be reset to the initial default.
- `reset_camera=Camera.ISO` / `Camera.TOP` / `Camera.BOTTOM` / `Camera.LEFT` / `Camera.RIGHT` / `Camera.FRONT` / `Camera.BACK` snaps to one of the axis-aligned camera presets — useful for reproducible screenshots.
