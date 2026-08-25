## show_all

Show every CAD object in the current scope (`locals()`) of the Python interpreter. Hosts with a visual debugger (e.g. ocp_vscode) call this at every debugger step.

### Command

```python
show_all(variables=None, exclude=None, classes=None, include=None, <keyword arguments>)
```

### Arguments

```text
    Show all variables in the current scope

    Parameters:
        variables:     Only show objects with names in this list of variable names,
                       i.e. do not use all from locals()
        exclude:       List of variable names to exclude from "show_all"
        classes:       Only show objects which are instances of the classes in this list
        include:       List of variable names that should be shown even though they
                       would be filtered out by `classes`. Only takes effect when
                       `classes` is set.

    Keywords for show_all:
        Valid keywords for "show_all" are the same as for "show"
```

This includes the [host keywords](show.md#host-keywords) (`port`, `viewer`, `anchor`, `cad_width`, `height`, `pinning`) — which of them your viewer accepts is the same as for `show`.

For more detail, see [show](show.md)

### Example

```python
from build123d import *

from ocp_viewer import *           # if using ocp_viewer
# from ocp_vscode import *         # if using ocp_vscode
# from jupyter_cadquery import *   # if using jupyter_cadquery
# from build123d_studio import *   # if using build123d_studio

set_defaults(axes=True, transparent=False, collapse=1, grid=(True, True, True))

box = Box(1, 2, 1)
chamfer(box.edges(), 0.4)
sphere = Sphere(0.8)

show_all(
    collapse="1",
    ortho=False
)
```
