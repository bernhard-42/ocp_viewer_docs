# Importing

All four viewer packages export the same Python API: every function, class and enum on these pages — `show`, `set_defaults`, `Camera`, `ColorMap`, `Animation`, all of them — is available under the same name in `ocp_vscode`, `ocp_viewer`, `jupyter_cadquery` and `build123d_studio`. That leaves two equally valid ways to write the import line.

## From your viewer's package

The classic form names the viewer you are working with:

=== "ocp_viewer"

    ```python
    from ocp_viewer import show, set_defaults, Camera
    ```

=== "ocp_vscode"

    ```python
    from ocp_vscode import show, set_defaults, Camera
    ```

=== "jupyter_cadquery"

    ```python
    from jupyter_cadquery import show, set_defaults, Camera
    ```

=== "build123d_studio"

    ```python
    from build123d_studio import show, set_defaults, Camera
    ```

This is the natural choice when a script belongs to one viewer — the import line documents where the objects will appear, and nothing else in the script changes if you ever move it to another viewer.

=== "ocp_viewer"

    ```python
    from ocp_viewer import *
    ```

=== "ocp_vscode"

    ```python
    from ocp_vscode import *
    ```

=== "jupyter_cadquery"

    ```python
    from jupyter_cadquery import *
    ```

=== "build123d_studio"

    ```python
    from build123d_studio import *
    ```

works as well: each package declares its public surface, so the star brings in exactly the documented names.

## The portable import

Scripts that should run unchanged against any viewer import from `ocp_viewer_core.viewer` instead:

```python
from ocp_viewer_core.viewer import show, set_defaults, Camera
```

or

```python
from ocp_viewer_core.viewer import *
```

This module looks at the environment, picks the viewer that environment belongs to, and offers that viewer's complete API — the same names, bound to the same objects, as importing from the viewer's own package:

| Environment                   | Resolves to        |
| ----------------------------- | ------------------ |
| A VS Code terminal or session | `ocp_vscode`       |
| A Jupyter CadQuery kernel     | `jupyter_cadquery` |
| build123d Studio              | `build123d_studio` |
| Anywhere else                 | `ocp_viewer`       |

It announces its choice once, on import, e.g. `using ocp_vscode`

`from ocp_viewer_core.viewer import *` is the star form; it is identical to the resolved viewer's own star import. The resolved viewer's package must be installed — the module selects, it does not install.

## Deep imports

For backwards compatibility, each viewer package keeps its historical submodules, so imports like `from ocp_vscode.config import set_defaults` or `from ocp_vscode.utils import ignore_camera_warnings` continue to work. These paths are part of each viewer's own layout and are not portable — a multi-viewer script uses the flat portable form above, which carries every name the deep paths do.

The `ocp_viewer_core` package itself also has submodules (`config`, `colors`, `animation`, …). They hold the shared vocabulary — the enums, the colormap classes, the camera warnings — and are mainly interesting for advanced use: functions that talk to a viewer, like `show` or `set_defaults`, exist only in a viewer's package and in `ocp_viewer_core.viewer`, never there.
