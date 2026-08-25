# The viewer

Every host embeds the same viewer window — the JavaScript half of `ocp-viewer-core`, rendering with [three-cad-viewer](https://github.com/bernhard-42/three-cad-viewer). Whether it sits in a VS Code panel, a browser tab, a Jupyter cell or the Studio app, the window looks the same and behaves the same. This page describes it from the user's side: what is on the screen and how to operate it with mouse and keyboard.

Almost everything described here can also be driven from Python: the initial state via keywords on the `show*` commands (see [show](show.md)), and a running viewer via [set_viewer_config](set_viewer_config.md). The UI and the Python keywords are two handles on the same state — a toggle you click in the toolbar is the same setting as the keyword of the same name.

## Layout

The window has four areas:

- the **canvas**, where the CAD objects are rendered. Before the first `show`, it displays the OCP splash logo.
- the **tree panel** on the left, with five tabs: the navigation tree and the Clip, Zebra, Material and Studio tools — see [Tabs](#tabs).
- the **info box** below the tree panel. On startup it reports the viewer version and control mode; later it logs what you pick — see [The info box](#the-info-box).
- the **toolbar** above the canvas, with view toggles, camera presets and tools.

The tree panel and the info box each have a small collapse toggle (**Tools** / **Info**) to fold them away. In **glass mode** (`glass=True`) the tree floats as a transparent overlay on the canvas instead of occupying its own column, giving the CAD object the full width. `tools=False` hides the toolbar and tree entirely. The viewer follows the `theme` setting: `"light"`, `"dark"`, or `"browser"` to adopt the surface's preference.

When the window is too narrow for the full toolbar, button groups collapse behind `…` buttons; clicking one unfolds its group.

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

| Action              | Binding                                                    |
| ------------------- | ---------------------------------------------------------- |
| Pick element        | double-click on the object, or click its label in the tree |
| Isolate element     | `shift` + double-click, or `shift` + click on a tree label |
| Hide element        | `meta` + double-click, or `meta` + click on a tree label   |
| Hide other elements | `shift` + `meta` + click on a tree label                   |
| Set camera target   | `shift` + `meta` + double-click on the object              |

Picking an element highlights it, draws its axis-aligned bounding box, and writes its path, name and bounding box into the info box. Setting the camera target re-aims the camera at the picked element's bounding-box center (also logged in the info box) — useful for rotating around a detail instead of the whole assembly.

## The navigation tree

One node per object or assembly level. Each leaf carries two toggle icons: the first for the object's **faces**, the second for its **edges** — so an object can be shown as solid, as wireframe (edges only), or not at all. Clicking an icon on a group node toggles everything below it; a group whose children differ shows a mixed state.

Four small buttons above the tree control its shape, matching the `collapse=` keyword: `1` collapses all nodes with a single leaf (`Collapse.LEAVES`), `R` expands the root only (`Collapse.ROOT`), `C` collapses everything (`Collapse.ALL`) and `E` expands everything (`Collapse.NONE`).

Clicking a node's **label** (not its icons) picks the object, with the `shift`/`meta` combinations from the table above.

## The toolbar

From left to right, in groups:

**View toggles**

- **Axes** — show the axes helper through the object's center (`axes`)
- **Axes at (0,0,0)** — move the axes helper to the origin (`axes0`)
- **Grid** — toggle all grids; the button's dropdown switches the XY, XZ and YZ planes individually (`grid`)
- **Perspective** — switch between orthographic and perspective camera (`ortho`)
- **Transparent** — render all faces semi-transparent (`transparent`, opacity from `default_opacity`)
- **Black edges** — draw all edges black instead of their own color (`black_edges`)

**Camera**

- **Reset view** — reset camera position, rotation, zoom and target
- **Resize** — fit the current objects into the canvas
- **Iso / Front / Back / Top / Bottom / Left / Right** — snap the camera to a preset (the `Camera` presets in Python)

**Tools** (one active at a time)

- **Explode** — pull an assembly apart to inspect its structure; a slider bar appears below the toolbar — see [Explode](#explode)
- **Distance** — measure between two picked shapes — see [Measure and select](#measure-and-select)
- **Properties** — read the properties of one picked shape
- **Select** — collect element indices for use in code

While a Distance, Properties or Select tool is active, a **filter dropdown** (All / Vertex / Edge / Face / Solid) appears in the toolbar to restrict what the mouse can pick.

**Right side**

- **Pin as PNG** — freeze the current view as a static image, on hosts that support pinning (e.g. Jupyter cell output)
- **Help** — the keyboard and mouse binding overlay (dismiss with `ESC`)

## The info box

The scrollable log under the tree panel. It shows, newest first:

- on startup: the three-cad-viewer version and the active control mode (trackball/orbit)
- on pick: the element's tree path and name, and its bounding box as a min/max/center table per axis
- on set-camera-target: the new target coordinates

Fold it away with the **Info** toggle when you need the vertical space.

## Tabs

The tree panel's header switches between five tabs. From Python, select one with `tab=` on any `show*` command or `set_viewer_config(tab=...)`, using the [UiTab](enums.md) enum. Each tab has an `R` button to reset its settings to the defaults.

### Clip

Three clipping planes — red, green and blue — cut the model open. Per plane:

- a **slider** with a numeric input moves the plane along its normal
- the current **normal** is displayed as `N1 = (x, y, z)`
- the **plane button** sets the plane's normal to the current view direction, so you cut exactly what you are looking at

Below the sliders:

- **Intersection** — cut only where all planes cut, instead of where any plane cuts (`clip_intersection`)
- **Planes** — show the clipping plane helpers (`clip_planes`)
- **Use object color caps** — fill the cut faces with the object's own color instead of red/green/blue (`clip_object_colors`)

The Python keywords are `clip_slider_0/1/2` and `clip_normal_0/1/2`.

### Zebra

Zebra stripe analysis for judging surface quality and continuity: stripes are projected onto the model, and the way they flow across face boundaries reveals tangency and curvature breaks that flat shading hides.

- **Stripe Count** (2–50, `zebra_count`) and **Stripe Opacity** (0–1, `zebra_opacity`)
- **Direction** — rotate the stripe pattern by 0–90° (`zebra_direction`)
- color scheme **B/W**, **Gray** or **Colors** (`zebra_color_scheme`)
- mapping **Reflection** (stripes follow what a mirror finish would reflect — the classic car-body check, view-dependent) or **Normal** (stripes follow the surface normals, view-independent) (`zebra_mapping_mode`)

### Material

The lighting and material of the plain CAD view (not Studio mode), all in percent: **Ambient Light**, **Direct Light**, **Metalness** and **Roughness** — the `ambient_intensity`, `direct_intensity`, `metalness` and `roughness` keywords.

### Studio

Photo-realistic rendering. A spinner shows while an HDR environment downloads.

- **Environment** — the HDR map lighting the scene, grouped into studio presets (Procedural Studio, Soft Light, High Contrast Studio, Bright Neutral, Clean Softbox, Spotlit Setup, Controlled Light, Hard Contrast Light) and outdoor presets (Urban Overcast, Outdoor Warm, Neutral Industrial, San Giuseppe Bridge). From Python a custom HDR URL is also accepted (`studio_environment`).
- **Use 4K maps** — sharper reflections, slower download (`studio_4k_env_maps`)
- **Env Intensity** (0–300%) and **Env Rotation** (0–360°) — brightness and orientation of the environment lighting
- **Background** — Environment, Transparent, Gradient Grey, Gradient Dark Grey, Solid White, Solid Grey, Solid Dark Grey (`studio_background`)
- **Tone Mapping** — PBR Neutral, ACES Filmic, or Linear (`studio_tone_mapping`), with **Exposure** (0–300%, `studio_exposure`)
- **Shadow Intensity** and **Shadow Softness** (`studio_shadow_intensity`, `studio_shadow_softness`)
- **AO Intensity** — screen-space ambient occlusion, darkening crevices and contact areas (`studio_ao_intensity`)
- **Texture Mapping** — Triplanar or Parametric UV projection for materials without UV coordinates (`studio_texture_mapping`)

The tab's `E` button opens the **material editor** for the selected object: double-click an object, press `E`, and a floating window shows the object's path and its PBR material values. Edit them live; changed values are marked red, `R` restores the original material, `X` closes the window. The red values are meant to be carried back into Python code as `override(...)` arguments — see [Materials and Studio mode](pbr_studio.md).

## Measure and select

The Distance, Properties and Select tools share the picking mechanics:

- `v` / `e` / `f` / `s` on the keyboard (or the filter dropdown) restrict picking to vertices / edges / faces / solids; `n` clears the filter
- `ESC` clears all selections; `backspace` or right-click removes only the last one

The readouts come from the host's measurement backend, which computes on the exact BRep geometry — the numbers are CAD-exact, not read off the tessellated mesh.

### Distance

Pick two shapes; an arrow with a result panel appears. Holding `shift` while picking the **second** shape measures center-to-center instead of the minimum distance — the panel says `(min)` or `(center)` accordingly. The panel shows:

- **distance**, and its **X | Y | Z** components
- **point 1** and **point 2**, the exact points measured between
- when both picks are edges or faces, the **angle** between them, with a reference row per pick saying what was compared: `line` or `tangent at P1/P2` for edges, `face normal` or `surface normal at P1/P2` for faces, plus the direction/normal vectors. For an edge against a face the angle is given relative to the surface.

### Properties

Pick one shape; the panel reports what the geometry kind warrants:

- **Vertex** — its `xyz` position
- **Edge** — by curve type: a line's start/middle/end; a circle's center and `radius / diam`; an ellipse's center and major/minor `radius / diam`; hyperbola/parabola vertex and endpoints; plus always the exact **length** and, where computable, the tangent angles to the XY plane at both ends
- **Face** — by surface type: a plane's center; a cylinder's center and radius; a cone's base radius and half angle; a sphere's radius; a torus's major/minor radius; a revolution's axis; plus always the exact **area** and the mid-face normal's angle to XY
- **Solid / Compound** — the exact **volume**
- everything except vertices also gets its **bounding box** (min, center, max, size)

### Select

Pick vertices, edges or faces — of one kind, on one solid — and their indices are collected and handed to the host; in ocp_vscode they land in the clipboard as a comma-separated list. Feed them to `select_vertices` / `select_edges` / `select_faces` to get the same elements back as build123d objects in code — see [Object selection](selector.md).

## Explode

**Explode** (`explode=True`) animates an assembly apart, each part moving away from the center. The animation control bar appears — scrub the timeline slider to any intermediate state. Explode is mutually exclusive with the analysis tools.

## The animation bar

A control bar with a timeline slider and play / pause / stop buttons, shown below the canvas whenever there is something to play: an animation sent from Python (see [Animation](animation.md)) or the explode animation. The slider scrubs; from Python, `set_relative_time(fraction)` does the same, which is how `save_as_gif` renders frame by frame.

## Small indicators

- **`⊢⊣` tick size** — the current grid spacing in model units, updated as the grid rescales with zoom, so you can always read distances off the grid.
- **Status line** — short hints above the canvas while a tool is active.
- **Warning banner** — temporary warnings across the top of the canvas (e.g. a material name that could not be resolved).

## What the viewer remembers

The viewer holds its state — camera, toggle settings, tree visibility, active tab — until the next `show`. What happens then is governed by the [config system](config.md): camera behavior by `reset_camera` (keep, recenter, or reset), and the toolbar-tier settings survive as the middle layer of the configuration precedence, so a toggle you clicked is not silently undone by the next `show` unless that show sets it explicitly.
