# The viewer

Every host embeds the same viewer window — the JavaScript half of `ocp-viewer-core`, rendering with [three-cad-viewer](https://github.com/bernhard-42/three-cad-viewer). Whether it sits in a VS Code panel, a browser tab, a Jupyter cell or the Studio app, the window looks the same and behaves the same. This page describes it from the user's side: what is on the screen and how to operate it with mouse and keyboard.

Almost everything described here can also be driven from Python: the initial state via keywords on the `show*` commands (see [show](show.md)), and a running viewer via [set_viewer_config](set_viewer_config.md). The UI and the Python keywords are two handles on the same state — a toggle you click in the toolbar is the same setting as the keyword of the same name.

## Layout

The window has three areas:

- the **canvas**, where the CAD objects are rendered. Before the first `show`, it displays the OCP splash logo.
- the **navigation tree** on the left, one node per object or assembly level, with visibility toggles.
- the **toolbar** above the canvas, with view toggles, camera presets and tools.

In **glass mode** (`glass=True`) the tree floats as a transparent overlay on the canvas instead of occupying its own column, giving the CAD object the full width. The tree panel can also be switched between tabs — see [Tabs](#tabs) below. `tools=False` hides the toolbar and tree entirely.

## Mouse navigation

| Action              | Binding                                            |
| ------------------- | -------------------------------------------------- |
| Rotate              | left mouse button, drag                            |
| Rotate up / down    | `ctrl` + left mouse button                         |
| Rotate left / right | `meta` (⌘ / Win) + left mouse button               |
| Pan                 | `shift` + left mouse button, or right mouse button |
| Zoom                | mouse wheel, or middle mouse button                |

Two control styles exist: **trackball** (the default — free rotation, no fixed up axis) and **orbit** (the up axis stays up). Choose with `orbit_control=True/False` on `show`. The rotate, pan and zoom sensitivities are the `rotate_speed`, `pan_speed` and `zoom_speed` settings.

## Mouse selection

| Action              | Binding                                                                                      |
| ------------------- | -------------------------------------------------------------------------------------------- |
| Pick element        | double-click on the object, or click its label in the tree — shows its bounding box and info |
| Isolate element     | `shift` + double-click, or `shift` + click on a tree label                                   |
| Hide element        | `meta` + double-click, or `meta` + click on a tree label                                     |
| Hide other elements | `shift` + `meta` + click on a tree label                                                     |
| Set camera target   | `shift` + `meta` + double-click on the object                                                |

## The navigation tree

Each leaf carries two toggles: one for the object's faces and one for its edges — so an object can be shown as solid, as wireframe, or not at all. Clicking a group node toggles everything below it; a mixed state is shown when some children are visible and some are not.

Four small buttons control the tree shape, matching the `collapse=` keyword: `1` collapses all nodes with a single leaf (`Collapse.LEAVES`), `R` expands the root only (`Collapse.ROOT`), `C` collapses everything (`Collapse.ALL`) and `E` expands everything (`Collapse.NONE`).

## The toolbar

From left to right:

- **Axes** / **Axes at (0,0,0)** — show the axes helper, either through the object's center or through the origin (`axes`, `axes0`)
- **Grid** — a dropdown to switch the XY, XZ and YZ grid planes individually (`grid`)
- **Perspective** — toggle between orthographic and perspective camera (`ortho`)
- **Transparent** — render all faces semi-transparent (`transparent`, opacity from `default_opacity`)
- **Black edges** — draw all edges black instead of their own color (`black_edges`)
- **Reset view** — reset camera position, rotation, zoom and target
- **Fit** — resize the view so the current objects fill the canvas
- **Iso / Front / Back / Top / Bottom / Left / Right** — snap the camera to a preset (the `Camera` presets in Python)
- **Explode** — pull an assembly apart along a slider to inspect its structure (`explode`)
- **Distance / Properties / Select** — the analysis tools, mutually exclusive with each other and with explode; see [Measure mode](measure.md) and [Object selection](selector.md)
- **Pin as PNG** — freeze the current view as a static image, on hosts that support pinning (e.g. Jupyter cell output)
- **Help** — show the keyboard and mouse binding overlay (dismiss with `ESC`)

## Tabs

The panel that holds the navigation tree has further tabs. From Python, select one with `tab=` on any `show*` command or `set_viewer_config(tab=...)`, using the [UiTab](enums.md) enum.

- **Tree** — the navigation tree described above.
- **Clip** — three clipping planes, each with a slider for its offset and an editable normal, plus a button per plane to align it with the current view direction. Options: intersection clipping (cut only where all planes cut), showing the helper planes, and filling the cut faces with the object's color instead of red/green/blue (`clip_*` keywords).
- **Zebra** — zebra stripe analysis for judging surface continuity: stripe count, opacity, direction, color scheme, and whether stripes follow reflections or surface normals (`zebra_*` keywords).
- **Material** — edit the material of the selected object (double-click an object, then press its `E` button). Changed values are marked so they can be carried back into Python code as overrides — see [Materials and Studio mode](pbr_studio.md).
- **Studio** — photo-realistic rendering: HDR environment maps, background style, tone mapping, exposure, shadows and ambient occlusion (`studio_*` keywords).

## Analysis tool keyboard shortcuts

Active in the Distance, Properties and Select tools:

- `v` / `e` / `f` / `s` — restrict picking to vertices / edges / faces / solids; `n` — no filter
- `ESC` — clear all selections
- `backspace` or right-click — remove only the last selection
- In distance measurement: `shift` + click for the second selection measures to the center instead of the minimum distance

## What the viewer remembers

The viewer holds its state — camera, toggle settings, tree visibility, active tab — until the next `show`. What happens then is governed by the [config system](config.md): camera behavior by `reset_camera` (keep, recenter, or reset), and the toolbar-tier settings survive as the middle layer of the configuration precedence, so a toggle you clicked is not silently undone by the next `show` unless that show sets it explicitly.
