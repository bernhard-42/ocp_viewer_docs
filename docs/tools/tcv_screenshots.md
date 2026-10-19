# Batch screenshots

[tcv_screenshots](https://github.com/bernhard-42/tcv_screenshots) renders CAD models to PNG files without a viewer window: it runs your example files, collects the screenshots they request and draws each one in a headless Chromium with the CAD Viewer. It is meant for documentation images and CI jobs - every screenshot is reproducible from code, and nothing has to be open or clicked.

It draws through the same pipeline as the viewers, so `screenshot` takes the same objects and keywords as [`show`](../show.md) and gives the same picture: colors, alphas, materials and studio mode, joints, clipping, camera settings. Unlike the [headless screenshots of OCP Viewer](../viewers/ocp_viewer/headless.md), there is no server to start and no browser to manage: one command does everything.

## Installation

```bash
pip install tcv-screenshots
playwright install chromium   # only needed without an installed Chrome or Edge
```

tcv_screenshots renders in an installed Google Chrome or Microsoft Edge when there is one; otherwise `playwright install chromium` downloads Playwright's own Chromium, see [Browser](#browser). Install the CAD library your examples use (build123d, CadQuery) into the same environment.

!!! info "Version"

    This page describes tcv_screenshots 1.0.0. 0.3.0 used a config dict of renderer names instead of the `show` keywords; see the migration notes in the [README](https://github.com/bernhard-42/tcv_screenshots#migration-from-030).

## Writing examples

An example is a Python file with a `main()` function that requests screenshots with `screenshot` and returns `get_screenshots()`:

```python
from build123d import *

part = fillet(Box(30, 20, 10).edges().filter_by(Axis.Z), 3) - Cylinder(4, 10)


def main():
    from tcv_screenshots import screenshot, get_screenshots

    screenshot(part, name="part", cad_width=800, height=600)
    return get_screenshots()
```

This writes `part.png`. An example can request several screenshots, e.g. the steps of an incremental construction, each under its own `name`:

```python
from build123d import *
from tcv_screenshots import Render, screenshot, get_screenshots

size = {"cad_width": 500, "height": 375}

box = Box(10, 10, 10)
screenshot(box, name="step1_box", **size)

filleted = fillet(box.edges(), 1)
screenshot(filleted, name="step2_fillet", **size)

final = filleted - Cylinder(2, 10)
screenshot(final, name="step3_hole", modes=[Render.FACES], **size)


def main():
    return get_screenshots()
```

`screenshot(*cad_objs, name, **kwargs)`:

- `cad_objs` - the objects to draw, like the positional arguments of `show`. Several objects are separate arguments; a list is drawn as one object
- `name` - the PNG file name, without `.png`
- `kwargs` - any keyword of [`show`](../show.md), e.g. `colors`, `alphas`, `modes`, `materials`, `render_joints`, `reset_camera`, `position` / `quaternion` / `target` / `zoom`, `tab="studio"` and the `studio_*` options. The [enums](../enums.md) (`Camera`, `Render`, `Collapse`, ...) can be imported from `tcv_screenshots`

The viewer-specific keywords `port`, `viewer`, `anchor` and `pinning` address a viewer window, which a screenshot does not have; they are ignored with a warning. `cad_width` and `height` set the image size.

### Defaults of a screenshot

| Keyword               | Default     |
| --------------------- | ----------- |
| `cad_width`           | 1200 (minimum 450) |
| `height`              | 800         |
| `theme`               | `"light"`   |
| `glass`               | `True`      |
| `tools`               | `False`     |
| `ambient_intensity`   | 1.0         |
| `direct_intensity`    | 1.1         |
| `metalness`           | 0.3         |
| `roughness`           | 0.65        |
| `default_edgecolor`   | `"#707070"` |
| `ortho`               | `True`      |
| `up`                  | `"Z"`       |
| `reset_camera`        | `Camera.RESET` |

Every screenshot starts with a fresh viewer: no setting of one screenshot (clipping, transparency, camera, tab) carries over to the next. The PNG's background is transparent.

## Running

```bash
# a single example, PNGs into the current directory
tcv_screenshots -f example.py

# a directory of examples
tcv_screenshots -d examples -o screenshots
```

Installing the package adds the `tcv_screenshots` command to the environment's `bin` folder (macOS, Linux) or `Scripts` folder (Windows, `tcv_screenshots.exe`). `python -m tcv_screenshots` does the same, e.g. when that folder is not on the `PATH`.

| Option                     | Meaning |
| -------------------------- | ------- |
| `-f`, `--file FILE`        | A single example file |
| `-d`, `--directory DIR`    | All `*.py` files of a directory, sorted by name (files starting with `_` or `.` are left out); requires `-o` |
| `-o`, `--output-folder DIR` | Where the PNG files go |
| `--no-gpu`                 | Render with the software renderer SwiftShader, see [GPU or SwiftShader](#gpu-or-swiftshader) |
| `--browser BROWSER`        | Use only `chrome`, `msedge` or `chromium` (Playwright's own), see [Browser](#browser) |
| `--check`                  | Show which browsers work, with version and renderer, and which one a run would use; renders nothing |
| `--no-headless`            | Show the browser window (for debugging) |
| `--pause`                  | Wait for Enter before each screenshot and print the camera values |
| `--debug MODELS_DIR`       | Save each model's data as JSON, show the browser console and full error messages |

Files that do not import `tcv_screenshots` are skipped. A failing example is reported with its traceback, starting at the line of the example that failed, and the other examples and models go on. The command exits with status 1 if any example or model failed, so a CI job fails as it should.

!!! tip "Finding a camera position"

    Run `tcv_screenshots -f example.py --no-headless --pause`, turn the model into position with the mouse, then press Enter. The printed `position`, `quaternion`, `target` and `zoom` can be passed to `screenshot` as they are.

## Browser

Every run uses the first browser that works: Microsoft Edge, Google Chrome, Playwright's Chromium on Windows (where Edge is always installed), and Google Chrome, Microsoft Edge, Playwright's Chromium on macOS and Linux. A browser that is not installed is skipped within milliseconds, so nothing is stored between runs. On Linux, Playwright looks for Chrome and Edge only at `/opt/google/chrome/chrome` and `/opt/microsoft/msedge/msedge`, where e.g. Google's `.deb` package installs Chrome; a Flatpak install (`flatpak run com.google.Chrome`) is not found, and the run goes on with the next browser. The browser in use is printed (`Browser: chrome 155.0...`).

```bash
tcv_screenshots --check
```

lists every candidate with its version and renderer and names the one a run would use; if none works, it says to run `playwright install chromium`. `--browser chrome|msedge|chromium` uses exactly one browser. An installed Chrome or Edge updates itself, so screenshots can change slightly with its versions; for reproducible screenshots, e.g. in CI, use `--browser chromium`, whose version only changes with Playwright's. Firefox and Safari cannot be used.

## GPU or SwiftShader

By default the browser renders with the GPU: Metal on macOS, Vulkan on Linux, Direct3D 11 on Windows (preferring a discrete GPU over the integrated one). The renderer in use is printed (`GPU rendering: ...`). `--no-gpu` renders with SwiftShader, a software renderer, which works without a GPU but is much slower for large models:

Rendering a heat exchanger (149 filleted tubes) and an assembly of 100 screws:

| Machine                 | SwiftShader | GPU                    |
| ----------------------- | ----------- | ---------------------- |
| Apple M1 Max, macOS     | 112.8 s     | 2.5 s (Metal)          |
| Ryzen 9 7900 + RTX 5060, Linux | 73.6 s | 2.5 s (Vulkan)   |
| Ryzen 9 7900 + RTX 5060, Windows 11 | 65.9 s | 2.4 s (Direct3D 11) |

GPU and SwiftShader images differ in anti-aliasing only, and different machines differ slightly too. A comparison against stored reference images should therefore always use the same renderer.

Without a usable GPU, Chromium falls back to SwiftShader and a warning says so. On CI runners, which have no GPU, pass `--no-gpu` to choose SwiftShader explicitly, and `--browser chromium` for a pinned browser version:

```yaml
- name: Generate screenshots
  run: |
    playwright install chromium
    tcv_screenshots -d examples -o screenshots --browser chromium --no-gpu
```

The repository's [workflow](https://github.com/bernhard-42/tcv_screenshots/blob/main/.github/workflows/screenshots.yml) runs this on Linux, macOS and Windows.

## Scripted use

The two steps of the command line are functions, for build scripts that need more control, e.g. the build of a documentation site. A build script and its examples could be laid out like this:

```text
build_screenshots.py
examples/
    box.py
    cylinder.py
more/
    extra_example.py
```

Each example is a file with a `main()` as in [Writing examples](#writing-examples). `examples/box.py`:

```python
from build123d import Box
from tcv_screenshots import get_screenshots, screenshot


def main():
    screenshot(Box(10, 20, 30), name="box")
    return get_screenshots()
```

`examples/cylinder.py`:

```python
from build123d import Cylinder
from tcv_screenshots import get_screenshots, screenshot


def main():
    screenshot(Cylinder(10, 30), name="cylinder")
    return get_screenshots()
```

`more/extra_example.py`, with two screenshots of the same part:

```python
from build123d import Box, fillet
from tcv_screenshots import Render, get_screenshots, screenshot

rounded = fillet(Box(20, 20, 20).edges(), 3)


def main():
    screenshot(rounded, name="rounded_box")
    screenshot(rounded, name="rounded_box_edges", modes=[Render.EDGES])
    return get_screenshots()
```

`build_screenshots.py` runs them, adds a screenshot of its own and renders everything:

```python
from build123d import Box
from tcv_screenshots import (
    collect_models,
    get_failed_models,
    get_screenshots,
    render_models,
    screenshot,
    set_defaults,
)

# Defaults for every following screenshot call, the keywords of `show`
set_defaults(cad_width=1000, height=1000)

# Run example files and directories of examples
collected = collect_models("examples", "more/extra_example.py")

# Or request screenshots directly, without example files
screenshot(Box(1, 2, 3), name="direct_box")
failed_screenshots = [name for name, _traceback in get_failed_models()]

rendered = render_models(collected.models + get_screenshots(), "screenshots")
print(rendered.written)                                         # the PNG files
print(collected.failed + failed_screenshots + rendered.failed)  # names of what failed
```

`python build_screenshots.py` writes `box.png`, `cylinder.png`, `rounded_box.png`, `rounded_box_edges.png` and `direct_box.png` into `screenshots/`, all 1000 x 1000 pixels, and prints an empty list of failures.

- `collect_models(*paths, models_dir=None)` returns `models` and `failed`. Each example runs with the defaults set before the call; a `set_defaults` inside one example does not reach the next. Screenshots requested before the call stay available to `get_screenshots()`
- `render_models(models, output_dir, *, headless=True, pause=False, debug=False, gpu=True, browser=None)` writes `<output_dir>/<name>.png` and returns `written` and `failed`. Where an event loop is already running (e.g. Jupyter), use `await render_models_async(...)` with the same arguments
- `screenshot` does not raise: a screenshot it cannot create is printed as `FAILURE` with its traceback, and `get_failed_models()` returns it. Neither function exits; use the returned names for the exit status of your script
- `set_defaults`, `get_defaults` and `reset_defaults` work as in the viewers, see [The config system](../config.md)

!!! info "Repeated parts"

    A part that appears in several screenshots of one run, also moved or rotated, is tessellated once and taken from the cache afterwards (ocp-tessellate 3.6.3 or newer, which tcv_screenshots 1.0.0 requires).
