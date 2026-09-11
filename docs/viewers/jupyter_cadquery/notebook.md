# Working in the notebook

What is particular about a viewer that lives inside a notebook: objects can display themselves, a click in the viewer can be read back into Python, and the notebook file remembers what its cells showed.

## Auto display

Importing `jupyter_cadquery` in a notebook kernel registers `show` as the display method of CadQuery's `Workplane`, `Shape`, `Assembly` and `Sketch` and of build123d's builders (`BuildPart`, `BuildSketch`, `BuildLine`), `Shape` and `ShapeList` — the import prints one line per library it took over. From then on an object as the last expression of a cell is shown, into the default sidecar or the cell, without writing `show`:

```python
from jupyter_cadquery import *

Box(1, 2, 3)          # shown
```

`show` with keywords is still there for everything auto display cannot say: names, colors, or any other [show keyword](../../show.md). `auto_show()` does the registration again by hand, for when something else has replaced the display method since — the libraries it covers are the ones installed in the environment, found when `jupyter_cadquery` was imported.

## Reading a pick back into Python

A double click on an object in the viewer selects it, and the viewer's `last_pick` property holds what was hit — the path in the navigation tree and the name. `get_pick(assembly, pick)` turns that into the CadQuery assembly member behind it:

```python
cv = show(assembly)
# double click a part in the viewer, then:
part = get_pick(assembly, cv.last_pick)
```

`get_pick` supports CadQuery assemblies; for anything else the tree path in `cv.last_pick["path"]` is what you have.

## What the notebook remembers

A cell viewer is a widget in the cell's output, and JupyterLab can save widget state with the notebook (*Settings → Save Widget State Automatically*). A notebook saved that way shows its viewers when converted to HTML with `jupyter nbconvert --to html`, without a kernel — see [Export](export.md). Without saved state, a reopened notebook shows the pinned PNG of a pinned cell (see [pinning](addressing.md#size)) and nothing for an unpinned one until the cell is run again.

A sidecar is not part of the notebook's outputs: it exists while the kernel runs, and a reopened notebook starts without it until the first `show`.
