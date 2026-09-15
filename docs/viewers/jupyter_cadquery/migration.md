# Migration from 4.x

What a notebook written for Jupyter CadQuery 4 meets in 5.1. The full list of changes is in the [changelog](https://github.com/bernhard-42/jupyter-cadquery/blob/master/CHANGELOG.md).

## Changed behaviour

- **A sidecar viewer fills its panel.** For a viewer of a fixed shape, as in 4.x, give `open_viewer` both `cad_width` and `height` — together they set the aspect ratio. See [Managing viewers](addressing.md#managing-viewers).
- **`show` keeps the camera.** The default is `reset_camera=Camera.KEEP` — unless your `~/.jcq_config` says otherwise, see below. The viewer warns when the new object is much larger or smaller than the view; `ignore_camera_warnings()` silences the warnings. For the 4.x behaviour use `set_defaults(reset_camera=Camera.RESET)`, or `reset_camera=Camera.RESET` per call. See [Keeping the camera orientation](../../reset_camera.md).
- **`ticks` defaults to 5** (was 10), as in the other viewers.
- **`modifier_keys` is one map per platform** and gains `alt`. On Windows and Linux, `meta` is the Alt key and `alt` the Windows key by default. See [Mouse and Keys](../../mouse_keys.md#modifier-keys).
- **The tree remembers what you hid.** Paths hidden or shown in the tree stay hidden or shown when the next `show` renders an object with the same paths.
- **`from jupyter_cadquery import *` still works.** For code that runs unchanged under every viewer of the family, import from `ocp_viewer_core.viewer` instead — it picks the viewer of the environment it runs in:

    ```python
    from ocp_viewer_core.viewer import *
    ```

## An existing `~/.jcq_config`

The file stores every setting it knows, so a `~/.jcq_config` written by 4.x still holds `reset_camera: reset`, `ticks: 10` and a three-key `modifier_keys`, and these stored values win over the new defaults. Delete the file — it is rewritten from the defaults on next use — or edit the three entries. See [Workspace Config](workspace_config.md).

## Removed and deprecated

- **`mate_scale`** is gone; use `helper_scale`.
- **`reset_camera=True`** is deprecated; use `Camera.RESET`.
- **`AnimationTrack` with `cv.add_track(...)` and `cv.animate(...)`** still works but warns. Animation is the shared `Animation` class now, as in every other viewer — see [Animation](../../animation.md):

    ```python
    from jupyter_cadquery import Animation

    animation = Animation()
    animation.add_track("/bottom/left_front", "rz", times, values)
    animation.animate(speed=3)
    ```

- **`ocp_vscode` is no longer installed** with Jupyter CadQuery. Its standalone viewer, `python -m ocp_vscode`, is the separate [OCP Viewer](../ocp_viewer/overview.md) package now.

## Back again

- **`export_html(filename, title=..., viewer=...)`** exports a viewer as a standalone page, see [Export](export.md).
- **`save_screenshot(filename, ...)`** saves the viewer as PNG, see [Export](export.md#png).
