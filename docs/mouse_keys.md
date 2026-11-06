# Mouse and Keys

## Mouse navigation

| Action              | Binding                                                           |
| ------------------- | ----------------------------------------------------------------- |
| Rotate              | left mouse button, drag                                           |
| Rotate up / down    | `ctrl` + left mouse button                                        |
| Rotate left / right | `meta` (⌘ on macOS, Alt on Windows and Linux) + left mouse button |
| Pan                 | `shift` + left mouse button, or right mouse button                |
| Zoom                | mouse wheel, or middle mouse button                               |

Two control styles exist: **trackball** (the default — free rotation, no fixed up axis) and **orbit** (the up axis stays up). Choose with `orbit_control=True/False` on `show`. The rotate, pan and zoom sensitivities are the `rotate_speed`, `pan_speed` and `zoom_speed` settings.

A rotate drag that starts on the model turns the camera around the point under the cursor, which stays where it is on screen. A drag that starts on the background turns around the centre of the bounding box of the visible objects. The mouse wheel, trackpad pinch and middle-button drag zoom towards the cursor (for a drag, towards where it started). The camera target moves with the pivot; **Reset** and **Recenter** put it back.

Two settings switch this off, separately: `cursor_pivot=False` rotates around the camera target, `zoom_to_cursor=False` zooms towards the centre of the view. Both default to `True`. They are arguments of `show` and `set_defaults`, `set_viewer_config` changes them on a running viewer, and every viewer stores them in its workspace config.

## Mouse selection

| Action              | Binding                                                    |
| ------------------- | ---------------------------------------------------------- |
| Pick element        | double-click on the object, or click its label in the tree |
| Isolate element     | `shift` + double-click, or `shift` + click on a tree label |
| Hide element        | `meta` + double-click, or `meta` + click on a tree label   |
| Hide other elements | `shift` + `meta` + click on a tree label                   |
| Set camera target   | `shift` + `meta` + double-click on the object, only with `cursor_pivot=False` |

Independent of any tool, the element under the cursor is continuously highlighted as the mouse moves, with a live mesh-based readout of its attributes; the toolbar's filter dropdown restricts what gets highlighted.

![live status](assets/viewer-status.png#only-light){ .center width="48%" }
![live status dark](assets/viewer-status-dark.png#only-dark){ .center width="48%" }

A double-click picks what the highlight shows, so the filter applies to it too: with "All", a double-click on or near an edge picks the edge (vertices come before edges, edges before faces), and with "Solid" it picks the whole solid. A double-click on something the filter excludes does nothing. `shift` and `meta` double-clicks act on the whole object the element under the cursor belongs to, and ignore the filter.

Picking highlights the element, draws the axis-aligned bounding box of the picked face or edge (a vertex gets a marker instead), and writes its path, name and bounding box (for a vertex, its coordinates) into the info box. A second double-click on the same element removes the box. A click on a tree label boxes the whole object.

![picking](assets/viewer-picking.png#only-light){ .center width="48%" }
![picking dark](assets/viewer-picking-dark.png#only-dark){ .center width="48%" }

`shift` + `meta` + double-click on an object depends on `cursor_pivot`:

- **`cursor_pivot=True` (the default):** it does nothing, and the viewer's help does not list it. A rotation centre need not be set by hand, because every rotate drag that starts on the model turns around the point under the cursor.
- **`cursor_pivot=False`:** it sets the camera target. The camera re-aims at the clicked point (at the object's bounding-box center if the viewer cannot read the point back), whatever the filter is set to, and later rotations turn around it. That is useful for rotating around a detail instead of the whole assembly.

`shift` + `meta` + click on a tree label (hide other elements) works the same either way.

## Modifier keys

The `shift` / `ctrl` / `meta` bindings above are not hard-wired: every viewer stores a `modifier_keys` mapping from the four logical keys (`shift`, `ctrl`, `meta`, `alt`) to the browser's modifier names (`shiftKey`, `ctrlKey`, `metaKey`, `altKey`), and the CAD Viewer reads its bindings from it. Remap them when a chord collides with your operating system or window manager.

The default depends on the operating system, because `metaKey` is ⌘ on macOS but the Windows/Super key on Windows and Linux, which the desktop takes for itself (Start menu, window snapping, moving windows) — so there the `meta` chords would never reach the viewer:

|                | `shift`    | `ctrl`    | `alt`                 | `meta`         |
| -------------- | ---------- | --------- | --------------------- | -------------- |
| macOS          | `shiftKey` | `ctrlKey` | `altKey`              | `metaKey` (⌘)  |
| Windows, Linux | `shiftKey` | `ctrlKey` | `metaKey` (Win/Super) | `altKey` (Alt) |

The stored setting holds both maps, `{"macOS": {...}, "default": {...}}`, and each viewer applies the one for the platform it runs on — so a synced or copied config works on every machine. A file written by an earlier release holds a single flat map and keeps it, on every platform; edit or regenerate the file to pick up the per-platform default.

| Viewer             | Where to set it                                                                                                                                                                           |
| ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| VS Code CAD Viewer | the `OcpCadViewer.view.modifier_keys` [setting](viewers/ocp_vscode/workspace_config.md), one map per platform (`macOS` and `default`), so a synced `settings.json` works on every machine |
| OCP Viewer         | `modifier_keys` in [`~/.ocpvscode_standalone`](viewers/ocp_viewer/workspace_config.md)                                                                                                    |
| Jupyter CadQuery   | `modifier_keys` in [`~/.jcq_config`](viewers/jupyter_cadquery/workspace_config.md)                                                                                                        |
| build123d Studio   | [Settings → Viewer](viewers/build123d_studio/workspace_config.md), Camera and controls                                                                                                    |
