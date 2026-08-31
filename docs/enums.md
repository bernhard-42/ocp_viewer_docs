# Enums reference

All enums below are exported by every viewer's package and accepted wherever the underlying string value would also work.

```python
from ocp_viewer_core.viewer import Camera, Collapse, Render, AnalysisTool, UiTab, StudioEnvironment, StudioBackground, StudioToneMapping, StudioTextureMapping
```

## `Camera`

Camera behavior for `reset_camera=` and the corresponding viewer setting.

| Member                                                        | Effect                                                                                    |
| ------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| `Camera.RESET`                                                | Reset position, rotation, zoom, target to defaults                                        |
| `Camera.CENTER`                                               | Keep position, rotation, zoom; recentre look-at on the new object                         |
| `Camera.KEEP`                                                 | Keep position, rotation, zoom, target — the default via the viewer settings. Warns when the new object may be invisible at the kept camera (silence with `ignore_camera_warnings()`) |
| `Camera.ISO`                                                  | Snap to isometric preset                                                                  |
| `Camera.TOP` / `BOTTOM` / `LEFT` / `RIGHT` / `FRONT` / `BACK` | Snap to the named axis-aligned preset                                                     |

## `Collapse`

Initial state of the navigation tree.

| Member            | Tree state                 |
| ----------------- | -------------------------- |
| `Collapse.NONE`   | Fully expanded             |
| `Collapse.LEAVES` | Collapse single-leaf nodes |
| `Collapse.ALL`    | All nodes collapsed        |
| `Collapse.ROOT`   | Only the root expanded     |

## `Render`

Per-object render mode used with `show_object(..., mode=...)` and `show(..., modes=[...])`.

| Member         | Shows         |
| -------------- | ------------- |
| `Render.ALL`   | Faces + edges |
| `Render.FACES` | Faces only    |
| `Render.EDGES` | Edges only    |
| `Render.NONE`  | Hidden        |

## `AnalysisTool`

Active analysis tool — see [measure.md](measure.md).

| Member                    | Tool                                              |
| ------------------------- | ------------------------------------------------- |
| `AnalysisTool.PROPERTIES` | Properties readout                                |
| `AnalysisTool.DISTANCE`   | Distance / angle measurement                      |
| `AnalysisTool.SELECT`     | Object selection — see [selector.md](selector.md) |
| `AnalysisTool.OFF`        | No analysis tool                                  |

Mutually exclusive with `explode=True`.

## `UiTab`

Side-panel tab selection — used by `set_viewer_config(tab=...)` and the `tab=` keyword on every `show*` command.

| Member           | Tab         |
| ---------------- | ----------- |
| `UiTab.TREE`     | Object tree |
| `UiTab.CLIP`     | Clipping    |
| `UiTab.ZEBRA`    | Zebra       |
| `UiTab.MATERIAL` | Material    |
| `UiTab.STUDIO`   | Studio      |

## Studio-mode enums

Used in `set_viewer_config(...)` and the `studio_*=` keywords on every `show*` command.

### `StudioEnvironment`

The HDR environment map lighting the scene. A custom HDR URL is also accepted in place of an enum member.

| Member | Environment |
| ------ | ----------- |
| `StudioEnvironment.PROCEDURAL_STUDIO` | Procedurally generated studio — the default, no download needed |
| `StudioEnvironment.SOFT_LIGHT` | Studio: soft light, neutral, backlight |
| `StudioEnvironment.HIGH_CONTRAST_STUDIO` | Studio: high contrast, softbox and ceiling lamp, crisp |
| `StudioEnvironment.BRIGHT_NEUTRAL` | Studio: white, product, bright, neutral lighting |
| `StudioEnvironment.CLEAN_SOFTBOX` | Studio: white, softbox, reflection, clean |
| `StudioEnvironment.SPOTLIT_SETUP` | Studio: lighting setup, spotlights |
| `StudioEnvironment.CONTROLLED_LIGHT` | Studio: product lighting, controlled, soft reflections |
| `StudioEnvironment.HARD_CONTRAST_LIGHT` | Studio: cyclorama, hard light, contrast |
| `StudioEnvironment.URBAN_OVERCAST` | Outdoor: urban, city, overcast |
| `StudioEnvironment.OUTDOOR_WARM` | Outdoor: dawn, warm, nature, sunrise |
| `StudioEnvironment.NEUTRAL_INDUSTRIAL` | Outdoor: warehouse, neutral, big space |
| `StudioEnvironment.SAN_GIUSEPPE_BRIDGE` | Outdoor: bridge, GPUOpen reference |

### `StudioBackground`

The scene background style.

| Member | Background |
| ------ | ---------- |
| `StudioBackground.ENVIRONMENT` | The environment map itself |
| `StudioBackground.TRANSPARENT` | Transparent |
| `StudioBackground.GRADIENT` | Gradient grey |
| `StudioBackground.GRADIENT_DARK` | Gradient dark grey |
| `StudioBackground.WHITE` | Solid white |
| `StudioBackground.GREY` | Solid grey |
| `StudioBackground.DARKGREY` | Solid dark grey |

### `StudioToneMapping`

The algorithm converting the HDR-lit scene to display colors.

| Member | Tone mapping |
| ------ | ------------ |
| `StudioToneMapping.NEUTRAL` | PBR Neutral |
| `StudioToneMapping.ACES` | ACES Filmic |
| `StudioToneMapping.NONE` | Linear (no tone mapping) |

### `StudioTextureMapping`

How textures are projected onto surfaces without UV coordinates.

| Member | Mapping |
| ------ | ------- |
| `StudioTextureMapping.TRIPLANAR` | Triplanar projection |
| `StudioTextureMapping.PARAMETRIC` | Parametric UV projection |
