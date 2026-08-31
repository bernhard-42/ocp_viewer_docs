## show_object

Incrementally show CAD objects in the viewer. The command supports the CQ-Editor parameters `obj`, `name` and `options` plus additional viewer specific args.

### Command

```python
show_object(obj, name=None, options=None, port=None, <keyword arguments>)
```

### Arguments

**Parameters**

| Parameter | Description |
| --------- | ----------- |
| `obj` | The CAD object to be shown |

**Keywords for `show_object`**

| Keyword | Description |
| ------- | ----------- |
| `name` | The name of the CAD object |
| `options` | A dict of color and alpha value: `{"alpha": 0.5, "color": (64, 164, 223)}`, with 0 <= alpha <= 1.0 and color as a 3-tuple of values between 0 and 255 |
| `parent` | Add another object, usually the parent of e.g. edges or vertices with alpha=0.25 |
| `clear` | In interactive mode, clear the stack of objects to be shown (typically used for the first object) |
| `update` | Update the object (remove old version) |
| `mode` | A [Render](enums.md#render) value for this object (default=None, i.e. `Render.ALL`).<br>`Render.ALL`: show faces and edges, `Render.EDGES`: show edges only, `Render.FACES`: show faces only, `Render.NONE`: hide object |
| `material` | Material object or material name string for this object (default=None) |
| `port` | The viewer to address when several are open — one of the viewer-specific keywords; see [Viewer-specific keywords](show.md#viewer-specific-keywords) for which viewer accepts which (port, viewer, anchor, cad_width, height, pinning) |
| `progress` | Show progress of tessellation, None is no progress indicator (default="-+*c").<br>Per object: "-": is reference, "+": gets tessellated with Python code, "*": gets tessellated with native code, "c": from cache |

The valid keywords to configure the viewer (`**kwargs`) are the same as for [show](show.md), including the [viewer-specific keywords](show.md#viewer-specific-keywords).

### Managing the object stack

- `reset_show()`

  Reset the object stack so the next `show_object` starts from a clean slate — typically at the top of a file that is executed repeatedly, to clean the object buffer.

- `show_object(obj, clear=True)`

  Restart a new object stack with this object.

For more detail, see [show](show.md)

### Example

```python
from ocp_viewer_core.viewer import show_object, reset_show, set_defaults, Collapse

import cadquery as cq

reset_show()  # use for repeated cell execution to clean object buffer

set_defaults(axes=True, transparent=False, collapse=Collapse.LEAVES, grid=(True, False, False))

box = cq.Workplane().box(1, 2, 1).edges().chamfer(0.4)
show_object(box, name="box", options={"alpha": 0.5})

sphere = cq.Workplane().sphere(0.6)

show_object(
    sphere,
    # show_object args
    "sphere",
    {"color": (10, 100, 110)},
    # viewer args
    ortho=False,
    zoom=1.25
)
```

![](./assets/show-show_object.png#only-light){.center width=50%}
![](./assets/show-show_object-dark.png#only-dark){.center width=50%}
