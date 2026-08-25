## show_object

Incrementally show CAD objects in the viewer. The command supports the CQ-Editor parameters `obj`, `name` and `options` plus additional viewer specific args.

### Command

```python
show_object(obj, name=None, options=None, port=None, <keyword arguments>)
```

### Arguments

```text
Parameters:
    obj:                     The CAD object to be shown

Keywords for show_object:
    name:                    The name of the CAD object
    options:                 A dict of color and alpha value: {"alpha":0.5, "color": (64, 164, 223)}
                                0 <= alpha <= 1.0 and color is a 3-tuple of values between 0 and 255
    parent:                  Add another object, usually the parent of e.g. edges or vertices with alpha=0.25
    clear:                   In interactive mode, clear the stack of objects to be shown
                                (typically used for the first object)
    update:                  Update the object (remove old version)
    mode:                    A Render value for this object (default=None, i.e. Render.ALL).
                                Render.ALL: show faces and edges
                                Render.EDGES: show edges only
                                Render.FACES: show faces only
                                Render.NONE: hide object
    material:                Material object or material name string for this object (default=None)
    port:                    The viewer to address when several are open — one of the host
                                keywords; see "Host keywords" in show.md for which viewer
                                accepts which (port, viewer, anchor, cad_width, height,
                                pinning)
    progress:                Show progress of tessellation with None is no progress indicator. (default="-+*c")
                                for object: "-": is reference,
                                            "+": gets tessellated with Python code,
                                            "*": gets tessellated with native code,
                                            "c": from cache
```

The valid keywords to configure the viewer (`**kwargs`) are the same as for [show](show.md), including the [host keywords](show.md#host-keywords).

### Typically useful parameters

- Reset the object stack:

  `reset_show()`

- Restart a new objects stack

  `show_object(obj, clear=True)`

For more detail, see [show](show.md)

### Example

```python
import cadquery as cq

from ocp_viewer import show_object, reset_show, set_defaults           # if using ocp_viewer
# from ocp_vscode import show_object, reset_show, set_defaults         # if using ocp_vscode
# from jupyter_cadquery import show_object, reset_show, set_defaults   # if using jupyter_cadquery
# from build123d_studio import show_object, reset_show, set_defaults   # if using build123d_studio

reset_show()  # use for repeated cell execution to clean object buffer

set_defaults(axes=True, transparent=False, collapse=1, grid=(True, True, True))

box = cq.Workplane().box(1, 2, 1).edges().chamfer(0.4)
show_object(box, name="box", options={"alpha": 0.5})

sphere = cq.Workplane().sphere(0.6)

show_object(
    sphere,
    # show_object args
    "sphere",
    {"color": (10, 100, 110)},
    # viewer args
    collapse="1",
    ortho=False
)
```
