## show_all

Show every CAD object in the current scope (`locals()`) of the Python interpreter. Hosts with a visual debugger (e.g. ocp_vscode) call this at every debugger step.

### Command

```python
show_all(variables=None, exclude=None, classes=None, include=None, <keyword arguments>)
```

### Arguments

**Parameters**

| Parameter | Description |
| --------- | ----------- |
| `variables` | Only show objects with names in this list of variable names, i.e. do not use all from `locals()` |
| `exclude` | List of variable names to exclude from `show_all` |
| `classes` | Only show objects which are instances of the classes in this list |
| `include` | List of variable names that should be shown even though they would be filtered out by `classes`. Only takes effect when `classes` is set |

Valid keywords for `show_all` are the same as for [show](show.md), including the [viewer-specific keywords](show.md#viewer-specific-keywords) (`port`, `viewer`, `anchor`, `cad_width`, `height`, `pinning`) — which of them your viewer accepts is the same as for `show`.

### Example

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

```python
from build123d import *

set_defaults(axes=True, center_grid=True, grid=(True, False, False))

box = Box(1, 2, 1)
chamfer(box.edges(), 0.4)
sphere = Sphere(0.8)

box.color = "red"

show_all(ortho=False)
```

![](./assets/show-show_all.png#only-light){.center width=50%}
![](./assets/show-show_all-dark.png#only-dark){.center width=50%}
