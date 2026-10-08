# Headless screenshots

## Using a headless browser

OCP Viewer renders in a browser. There is no server-side rendering: `save_screenshot` asks the browser to read its canvas back as a PNG, the browser sends the image to the server, and the server writes the file. A screenshot therefore always needs a browser page connected to the server — on a machine without a display, or in a script that should run unattended, that page comes from a headless browser.

Two ways to get one. Both end with the same two Python calls, so pick by how you want the browser's lifetime managed.

```python
from build123d import *
from ocp_viewer import Camera, show, save_screenshot

part = fillet(Box(30, 20, 10).edges().filter_by(Axis.Z), 3) - Cylinder(4, 10)

show(part, port=3999, reset_camera=Camera.RESET)
save_screenshot("part.png", port=3999)
```

`save_screenshot` returns once the file exists; it gives up after two seconds with a warning if no browser answered. The `port=` keyword addresses the server the headless page is attached to — see [Addressing a viewer](addressing.md). The PNG's background is transparent — see [Screenshots](../../api.md#screenshots) before flattening it with Pillow.

!!! warning "Camera"

    Pass `reset_camera=Camera.RESET` on the `show()` that precedes a screenshot. With the default `Camera.KEEP`, a `show()` issued right after the page has loaded can leave the model small and off-centre in the PNG, and the client prints a `CameraKeepWarning`.

!!! warning "Polling"

    Polling for the file to arrive currently waits for 2 sec only. This could be too short for large models. Try `polling=True` and if an error happens, set it to `False` and poll for the screenshot yourself.

- **Headless Chrome**

    A long-lived browser the scripts attach to again and again. Start the server, then point Chrome at its page:

    ```bash
    python -m ocp_viewer --port 3999 --theme light --no_tools --no_glass
    ```

    === "Windows"
        
        ```text
        "C:\Program Files\Google\Chrome\Application\chrome.exe" ^
            --headless=new --window-size=1600,1000 --hide-scrollbars ^
            --user-data-dir=%TEMP%\ocp-headless ^
            http://127.0.0.1:3999/viewer
        ```

    === "MacOS"

        ```text
        "/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" \
            --headless=new --window-size=1600,1000 --hide-scrollbars \
            --user-data-dir=/tmp/ocp-headless \
            http://127.0.0.1:3999/viewer
        ```

    === "Linux"

        ```text
        /opt/google/chrome/google-chrome \
            --headless=new --window-size=1600,1000 --hide-scrollbars \
            --user-data-dir=/tmp/ocp-headless \
            http://127.0.0.1:3999/viewer
        ```

    Headless Chrome renders WebGL on the machine's GPU without further flags. `--window-size` sets the size of the PNG. From then on every `show()` and `save_screenshot()` against port 3999 lands in that page, until the Chrome process is stopped.


- **Playwright**

    The browser starts and stops with the script, which suits tests and CI. `pip install playwright` installs the Python package and its driver only; no browser is downloaded. `channel="chrome"` in the script below drives the Google Chrome installed on the machine, which uses the GPU like the command above. Playwright looks for it in Chrome's standard location (`/Applications/Google Chrome.app` on macOS, `/opt/google/chrome/google-chrome` on Linux, `C:\Program Files\Google\Chrome\Application\chrome.exe` under the program files folders on Windows).

    ```bash
    pip install playwright
    ```

    ***Running Chrome installed on the system through Playwright***

    ```python
    from playwright.sync_api import sync_playwright
    from build123d import *
    from ocp_viewer import Camera, show, save_screenshot

    part = fillet(Box(30, 20, 10).edges().filter_by(Axis.Z), 3) - Cylinder(4, 10)

    with sync_playwright() as p:
        browser = p.chromium.launch(headless=True, channel="chrome")
        page = browser.new_page(viewport={"width": 1600, "height": 1000})
        page.goto("http://127.0.0.1:3999/viewer")
        page.wait_for_selector("#cad_viewer canvas")

        show(part, port=3999, reset_camera=Camera.RESET)
        save_screenshot("part.png", polling=False, port=3999)

        browser.close()
    ```

    The server (`python -m ocp_viewer --port 3999`) still has to be running; Playwright only replaces the Chrome command above. `page.screenshot(path=...)` works as well and captures the whole page including the axes gizmo, but the model still arrives through `show()`, so it adds nothing over `save_screenshot`.

    Without a system Chrome, let Playwright download its own Chromium. Check out the Playwright docs how to do this.

!!! warning "No GPU available"

    On a machine without a GPU — a CI runner, a container, a bare server — add Chrome's software renderer:

    ```
    --use-angle=swiftshader --enable-unsafe-swiftshader --ignore-gpu-blocklist
    ```

    to the headless Chrome command, or as `args=[...]` to `p.chromium.launch(...)`. Rendering is then a few times slower but produces the same picture. Measured on an Apple Silicon Mac, by asking the viewer page for its WebGL renderer and timing `show()` plus `save_screenshot()`:

    | Binary | Flags | Renderer | show + screenshot |
    |---|---|---|---|
    | installed Google Chrome | none | ANGLE Metal, Apple GPU | 0.11 s |
    | installed Google Chrome | SwiftShader flags | SwiftShader | 0.53 s |
    | Playwright `channel="chrome"` | none | ANGLE Metal, Apple GPU | 0.12 s |
    | Playwright `channel="chrome"` | SwiftShader flags | SwiftShader | 0.64 s |
    | Playwright's bundled Chromium | none | SwiftShader | 1.58 s |
    | Playwright's bundled Chromium | `--use-angle=metal --ignore-gpu-blocklist` | ANGLE Metal, Apple GPU | 0.22 s |

    The GPU and SwiftShader screenshots differ in a few pixels, not in content. The table has not been reproduced on Linux or Windows, where the GPU backend name differs and a display or a Vulkan driver may be needed.

## When the viewer is not needed

If the question is only whether some build123d or CadQuery code produces the shape you meant, [tcv-screenshots](https://github.com/bernhard-42/tcv_screenshots) renders a PNG with one command and no server:

```bash
pip install tcv-screenshots
playwright install chromium
python -m tcv_screenshots -f example.py -o screenshots
```

It bundles its own copy of three-cad-viewer and does not involve OCP Viewer, so it cannot show how a model looks in a running viewer with its workspace config applied.
