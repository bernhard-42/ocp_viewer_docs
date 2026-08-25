## push_object and show_objects

Incrementally show CAD objects in the viewer. The commands support the CQ-Editor parameters `obj`, `name` and `options` plus additional viewer-specific args. Compared to `show_object`, this pair is the lazy variant: `push_object` collects objects without rendering, and `show_objects` then renders them in a single batch — much faster than calling `show_object` repeatedly.

### Command

```python
push_object(obj, name=None, color=None, alpha=None, material=None, mode=None, clear=False, update=False)
```

### Arguments

    Parameters:
        obj: The object to be added or updated. Must have 'name', 'label', 'color', or 'alpha'
            attributes if corresponding arguments are not provided.
        name (str, optional): The name to associate with the object. If not provided,
            attempts to use 'name' or 'label' attribute of obj.
        color (any, optional): The color to associate with the object. If not provided,
            attempts to use 'color' attribute of obj.
        alpha (float, optional): The alpha (transparency) value for the object. If not provided,
            attempts to use 'alpha' attribute of obj, defaults to 1.0.
        material (PbrProperties or str, optional): Material object or material name string for the object.
        mode (Render, optional): A Render value for this object (Render.ALL, Render.EDGES, Render.FACES, Render.NONE).
        clear (bool, optional): If True, clears the OBJECTS registry before adding the new object.
        update (bool, optional): If True, updates an existing object with the same name;
            otherwise, appends as a new object.

    Raises:
        ValueError: If no name is provided and the object does not have a 'name' or 'label' attribute.

### Command

```python
show_objects(<keyword arguments>)
```

### Arguments

```text
    Keywords for show_objects:
        progress:                Show progress of tessellation with None is no progress indicator. (default="-+*c")
                                 for object: "-": is reference,
                                             "+": gets tessellated with Python code,
                                             "*": gets tessellated with native code,
                                             "c": from cache
```

The valid keywords to configure the viewer (`**kwargs`) are the same as for [show](show.md), including the [host keywords](show.md#host-keywords). Note that they belong to `show_objects` only: `push_object` just collects into the local registry, nothing reaches the viewer, so it takes no host keywords.

### Example

=== "ocp_viewer"

    ```python
    import cadquery as cq

    from ocp_viewer import push_object, show_objects, set_defaults

    set_defaults(axes=True, transparent=False, collapse=1, grid=(True, True, True))

    box = cq.Workplane().box(1, 2, 1).edges().chamfer(0.4)
    push_object(box, name="box", alpha=0.5)

    sphere = cq.Workplane().sphere(0.6)
    push_object(sphere, name="sphere", alpha=0.5)

    show_objects(
        collapse="1",
        ortho=False
    )
    ```

=== "ocp_vscode"

    ```python
    import cadquery as cq

    from ocp_vscode import push_object, show_objects, set_defaults

    set_defaults(axes=True, transparent=False, collapse=1, grid=(True, True, True))

    box = cq.Workplane().box(1, 2, 1).edges().chamfer(0.4)
    push_object(box, name="box", alpha=0.5)

    sphere = cq.Workplane().sphere(0.6)
    push_object(sphere, name="sphere", alpha=0.5)

    show_objects(
        collapse="1",
        ortho=False
    )
    ```

=== "jupyter_cadquery"

    ```python
    import cadquery as cq

    from jupyter_cadquery import push_object, show_objects, set_defaults

    set_defaults(axes=True, transparent=False, collapse=1, grid=(True, True, True))

    box = cq.Workplane().box(1, 2, 1).edges().chamfer(0.4)
    push_object(box, name="box", alpha=0.5)

    sphere = cq.Workplane().sphere(0.6)
    push_object(sphere, name="sphere", alpha=0.5)

    show_objects(
        collapse="1",
        ortho=False
    )
    ```

=== "build123d_studio"

    ```python
    import cadquery as cq

    from build123d_studio import push_object, show_objects, set_defaults

    set_defaults(axes=True, transparent=False, collapse=1, grid=(True, True, True))

    box = cq.Workplane().box(1, 2, 1).edges().chamfer(0.4)
    push_object(box, name="box", alpha=0.5)

    sphere = cq.Workplane().sphere(0.6)
    push_object(sphere, name="sphere", alpha=0.5)

    show_objects(
        collapse="1",
        ortho=False
    )
    ```
