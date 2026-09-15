# Migration from 4.0

What a script written for OCP CAD Viewer 4.0 meets in 4.1. The full list of changes is in the [changelog](https://github.com/bernhard-42/vscode-ocp-cad-viewer/blob/main/CHANGELOG.md).

## Autostart with `ocp_viewer_core.viewer`

The viewer starts automatically when an open Python file contains one of the lines in `OcpCadViewer.advanced.autostartTriggers`. 4.1 adds the two lines of the viewer-neutral import to that list, so a file that begins with

```python
from ocp_viewer_core.viewer import *
```

starts the viewer like `from ocp_vscode import *` does. **If you have changed `autostartTriggers` in your settings, your list replaces the default and does not gain the two new entries** — add them yourself:

```json
  "OcpCadViewer.advanced.autostartTriggers": [
    "from ocp_vscode import",
    "import ocp_vscode",
    "from ocp_viewer_core.viewer import",
    "import ocp_viewer_core.viewer"
  ]
```

See [Workspace Config](workspace_config.md) and, for the import itself, [Importing](../../importing.md).

## Changed behaviour

- **`modifier_keys` is one map per platform.** `OcpCadViewer.view.modifier_keys` holds a `macOS` and a `default` map (Windows and Linux); the extension applies the one for the platform it runs on, so a synced `settings.json` serves every machine. On Windows and Linux, `meta` is the Alt key and `alt` the Windows key by default. A flat map, the setting's previous shape, is still taken as it is. See [Mouse and Keys](../../mouse_keys.md#modifier-keys).
- **Values are validated.** `reset_camera`, `collapse`, `theme`, `up`, `orbit_control` and the other enumerated keys are checked when `show()`, `set_defaults()` or `set_viewer_config()` is called — a typo raises instead of vanishing. Keywords that belong to another viewer (`viewer`, `anchor`, `pinning`, `cad_width`, `height`) are refused with a message.
- **`set_viewer_config()` applies the `studio_*` keys**, which the viewer used to drop, and `modifier_keys` can be set at runtime.
- **`ocp_vscode` depends on `ocp-viewer-core` only.** `ocp_tessellate`, `websockets`, `questionary`, `pillow` and `threejs-materials` come with the core; `flask`, `flask_sock`, `click`, `pyperclip`, `pyaml`, `requests` and `pygltflib` are no longer installed. The [Library Manager](managers.md) lists `ocp_viewer_core` so the core can be upgraded from the sidebar.
- **The extension and `ocp_vscode` must agree on major.minor**; a patch-level difference is accepted. The viewer page checks the core's JavaScript against the core's Python on every show and reports a mismatch to the developer console.
- **`from ocp_vscode import *` still works.** For code that runs unchanged under every viewer of the family, import from `ocp_viewer_core.viewer` instead — it picks the viewer of the environment it runs in (see the autostart note above):

    ```python
    from ocp_viewer_core.viewer import *
    ```

## Removed and deprecated

- **`OcpCadViewer.view.dark`** is gone; it had been deprecated in favour of `OcpCadViewer.view.theme`. A settings file still carrying `dark` gets the theme default (`browser`).
- **`control`** and **`mate_scale`** are gone; use `orbit_control` and `helper_scale`. Both had been deprecated.
- **`up="L"`** is no longer accepted; `up` is `"Z"` or `"Y"`.
- **`reset_camera=True` / `False`**, **`collapse="C"` / `"1"` / `"R"` / `"E"`**, **`render_edges`** and **`show_sketch_local`** still work but warn; use `Camera.RESET` / `Camera.CENTER`, the `Collapse` enum, `modes` and `show_locals`.
- **The standalone viewer** (`python -m ocp_vscode` without `--backend`) is the separate [OCP Viewer](../ocp_viewer/overview.md) package now; running the module without arguments says so.
