# Workspace Config

Jupyter CadQuery persists its settings in `~/.jcq_config`, a YAML file. It is the viewer-settings tier of the [config system](../../config.md): the starting values that `set_defaults()` overrides per Python process and show keywords per call.

## Reading and writing

- On first use, the viewer merges `~/.jcq_config` (if present) over its built-in defaults.
- `save_user_defaults()` writes the current workspace defaults back to `~/.jcq_config` — the way to persist a setup you have arrived at.
- `get_user_defaults()` returns the merged dict for inspection.
- The file is plain YAML and can be edited by hand; unknown keys are ignored.

```python
from jupyter_cadquery import set_defaults, save_user_defaults

set_defaults(glass=True, axes0=True, grid=(True, False, False))
save_user_defaults()
```

## Settings and defaults

The keys and their built-in defaults (aligned with the other viewers):

| Setting | Default | Description |
| ------- | ------- | ----------- |
| `theme` | `browser` | `light` / `dark` / `browser` |
| `glass` | `true` | Glass mode (tree overlays the canvas) |
| `tools` | `true` | Show toolbar |
| `tree_width` | `240` | Navigation tree width in px |
| `new_tree_behavior` | `true` | Eye icon controls the whole object, mesh icon only the wireframe — see [Tree](../../tabs.md#tree) |
| `control` | `trackball` | `trackball` or `orbit` mouse control |
| `modifier_keys` | shift/ctrl/meta/alt | Mapping to JS modifier key names, for the [mouse bindings](../../mouse_keys.md#modifier-keys) |
| `up` | `Z` | Up direction |
| `pan_speed`, `rotate_speed`, `zoom_speed` | `1` | Mouse speeds |
| `axes` | `false` | Show axes |
| `axes0` | `true` | Show axes at origin |
| `grid` | `[false, false, false]` | Grid planes XY / XZ / YZ |
| `center_grid` | `false` | Center grid at object/origin |
| `grid_font_size` | `12` | Grid axis label font size |
| `ticks` | `5` | Hint for grid tick count |
| `ortho` | `true` | Orthographic camera |
| `transparent` | `false` | Show objects transparent |
| `default_opacity` | `0.5` | Opacity for transparent objects |
| `black_edges` | `false` | Render edges in black |
| `collapse` | `leaves` | Tree initial state: `none` / `leaves` / `all` / `root` |
| `reset_camera` | `KEEP` | Camera behavior between `show` calls — see [Keeping the camera orientation](../../reset_camera.md) |
| `explode` | `false` | Turn explode mode on |
| `deviation` | `0.1` | Linear deviation for tessellation |
| `angular_tolerance` | `0.2` | Angular deflection for tessellation |
| `default_color` | `#e8b024` | Default shape color |
| `default_edgecolor` | `#707070` | Default edge color |
| `default_thickedgecolor` | `MediumOrchid` | Default thick-edge / line color |
| `default_facecolor` | `Violet` | Default face color |
| `default_vertexcolor` | `MediumOrchid` | Default vertex color |
| `ambient_intensity` | `1` | Ambient light intensity |
| `direct_intensity` | `1.1` | Direct light intensity |
| `metalness` | `0.3` | Material metalness |
| `roughness` | `0.65` | Material roughness |

Sidecar geometry (`cad_width`, `height`, `anchor`, `pinning`) is not a stored setting — a notebook cell decides it per `open_viewer` / `show` call, see [Addressing a viewer](addressing.md).
