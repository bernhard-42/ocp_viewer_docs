# Jupyter CadQuery API

The functions this viewer has beyond the [shared Python API](../../api.md). Everything shared — the `show` family, `set_defaults`, `set_viewer_config`, `status`, `save_screenshot`, animation, color maps — is documented once, in the shared chapters, and takes this viewer's `viewer=` keyword as described under [Addressing a viewer](addressing.md#addressing-a-viewer).

| function | see |
| --- | --- |
| `open_viewer(title, ...)` | [Managing viewers](addressing.md#managing-viewers) |
| `close_viewer(title)` | [Managing viewers](addressing.md#managing-viewers) |
| `close_viewers()` | [Managing viewers](addressing.md#managing-viewers) |
| `get_viewer(title=None)` | [Managing viewers](addressing.md#managing-viewers) |
| `get_viewers()` | [Managing viewers](addressing.md#managing-viewers) |
| `get_viewer_by_id(id)` | [Managing viewers](addressing.md#managing-viewers) |
| `get_viewers_by_id()` | [Managing viewers](addressing.md#managing-viewers) |
| `set_default_viewer(title, anchor=)` | [Addressing a viewer](addressing.md#addressing-a-viewer) |
| `get_default_viewer()` | [Addressing a viewer](addressing.md#addressing-a-viewer) |
| `export_html(filename, ...)` | [Export](export.md) |
| `auto_show()` | [Auto display](notebook.md#auto-display) |
| `get_pick(assembly, pick)` | [Reading a pick back](notebook.md#reading-a-pick-back-into-python) |
| `enable_replay(...)` | [Replay](replay.md), from `jupyter_cadquery.replay` |
| `disable_replay()` | [Replay](replay.md), from `jupyter_cadquery.replay` |
| `replay(obj, ...)` | [Replay](replay.md), from `jupyter_cadquery.replay` |
| `save_user_defaults()` | [Workspace Config](workspace_config.md) |
| `get_user_defaults()` | [Workspace Config](workspace_config.md) |
| `cvw_version` | the installed cad-viewer-widget version, the one an [export](export.md) loads |

`show` returns the `CadViewer` it drew into. Its properties (`zoom`, `position`, `quaternion`, `target`, `tab`, `axes`, `grid`, the clip and zebra settings, `last_pick`, …) read the viewer's live state and, where the viewer can change, set it — `cv.tab = "clip"` is `set_viewer_config(tab=UiTab.CLIP)` by another route.
