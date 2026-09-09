# Workspace Config

OCP Viewer takes its settings from three places, lowest to highest precedence:

1. built-in defaults
2. the config file `~/.ocpvscode_standalone` (if present)
3. CLI flags passed to `python -m ocp_viewer`

These form the viewer-settings tier of the [config system](../../config.md) — the starting values that `set_defaults()` overrides per Python process and show keywords per call.

## The config file `~/.ocpvscode_standalone`

Create it, populated with the built-in defaults, with:

```bash
python -m ocp_viewer --create_configfile
```

It is YAML, straightforward to edit by hand:

```yaml
theme: dark
tree_width: 300
axes: true
axes0: true
grid_xy: true
pan_speed: 0.5
rotate_speed: 1.0
zoom_speed: 0.5
```

Only keys that exist in the built-in defaults are honored; everything else is ignored. Delete the file to fall back to the built-in defaults entirely. (The file keeps its historic name — a user who has one should not have to move it because the package was renamed.)

## Settings and defaults

Every setting is also a CLI flag (`--tree_width 300`, boolean flags as `--axes` or `--no_glass`); `python -m ocp_viewer --help` lists them all.

| Setting | Default | Description |
| ------- | ------- | ----------- |
| `theme` | `browser` | `light` / `dark` / `browser` (follow the browser) |
| `debug` | `false` | Print what the server does; as a viewer setting it also reaches `show()`'s `debug` in every client |
| `no_glass` | `false` | Disable glass mode |
| `no_tools` | `false` | Hide the toolbar |
| `tree_width` | `240` | Navigation tree width in px |
| `new_tree_behavior` | `true` | Eye icon controls the whole object, mesh icon only the wireframe — see [Tree](../../tabs.md#tree) |
| `control` | `trackball` | `trackball` or `orbit` mouse control |
| `modifier_keys` | shift/ctrl/meta/alt | Mapping to JS modifier key names, for the [mouse bindings](../../mouse_keys.md#modifier-keys) |
| `up` | `Z` | Up direction: `Z` / `Y` / `L` (legacy) |
| `pan_speed`, `rotate_speed`, `zoom_speed` | `1` | Mouse speeds |
| `axes` | `false` | Show axes |
| `axes0` | `true` | Show axes at origin |
| `grid_xy`, `grid_xz`, `grid_yz` | `false` | Grid planes |
| `center_grid` | `false` | Center grid at object/origin |
| `grid_font_size` | `12` | Grid axis label font size |
| `ticks` | `5` | Hint for grid tick count |
| `perspective` | `false` | Perspective camera instead of orthographic |
| `transparent` | `false` | Show objects transparent |
| `default_opacity` | `0.5` | Opacity for transparent objects |
| `black_edges` | `false` | Render edges in black |
| `collapse` | `leaves` | Tree initial state: `none` / `leaves` / `all` / `root` |
| `reset_camera` | `KEEP` | Camera behavior between `show` calls — see [Keeping the camera orientation](../../reset_camera.md) |
| `explode` | `false` | Turn explode mode on |
| `deviation` | `0.1` | Linear deviation for tessellation |
| `angular_tolerance` | `0.2` | Angular deflection for tessellation |
| `default_color` | `#e8b024` | Default shape color (CSS3 names OK) |
| `default_edgecolor` | `#707070` | Default edge color |
| `default_thickedgecolor` | `MediumOrchid` | Default thick-edge / line color |
| `default_facecolor` | `Violet` | Default face color |
| `default_vertexcolor` | `MediumOrchid` | Default vertex color |
| `ambient_intensity` | `1` | Ambient light intensity |
| `direct_intensity` | `1.1` | Direct light intensity |
| `metalness` | `0.3` | Material metalness |
| `roughness` | `0.65` | Material roughness |

`--timeit` is a flag without a row: it is not stored in the file, but a server started with it answers `timeit: true` as a viewer setting, so every client's `show()` prints its Python and JavaScript timings until a `set_defaults(timeit=False)` or a show keyword says otherwise.

Only a flag you actually type counts as a choice: an option left at its default never overrides the file, whatever the built-in default happens to be.

Not viewer settings, read straight off the command line: `--port` and `--host` (where to listen), `--create_configfile`, and `--max_reconnect_attempts` (how long the page keeps trying to reconnect when the server goes away; `-1` for infinite).
