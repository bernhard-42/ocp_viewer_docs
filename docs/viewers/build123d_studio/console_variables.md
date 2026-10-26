# Console and variables

The bottom row of the window shows the kernel twice: as a transcript you can type at, and as a table you can open.

## The console

The **Console** tab is a `jupyter console` attached to the same kernel [Run Cell](running.md#on-the-kernel) sends to. Keystrokes go to a pty and its output is written back verbatim, so it is IPython: line editing, history search, tab completion, `?` and `??`, and the magics.

Run a cell with `Shift-Enter`, then ask the console about what it left behind:

```text
In [1]: part.volume
Out[1]: 1256.6370614359173
```

Ten thousand lines of scrollback are kept. Right-clicking offers copy and paste; on macOS `Cmd-C` / `Cmd-V` work through the application menu.

**About** has a **Kernel connection** section with this instance's Jupyter connection file and the command that uses it:

```
jupyter-console --existing "<the path About shows>"
```

Keep the quotes: the path contains a space on macOS and Windows. Install `jupyter-console` outside Studio; the environment here is managed by Settings.

!!! warning "Leave that console with `Ctrl-D`"

    Typing `exit` shuts down the kernel — Studio's kernel. `Ctrl-D` leaves it running. This is true of any console attached with `--existing`.

## The variable explorer

The pane beside the console lists what is in the namespace: name, type and a short `repr`, with a length in brackets for anything sized and build123d's `label` where an object has one. Drag the boundaries between the columns to re-proportion them; the widths are remembered.

The rows are refreshed whenever the kernel goes idle — after a run, after a line in the console. The listing reads only what is already there and computes no geometry.

Expanding a row asks for its contents at that moment, as deep as the data goes: rows inside rows are addressed by position, so an assembly unrolls by its children. Long collections are paged.

**Filter and sort.** The box above the table filters the rows: type part of a name — or of a build123d label — and only matching variables stay, through every refresh, until Escape clears it. Clicking **Name** or **Type** in the header sorts by that column; a second click reverses it, a third returns to the order the names were defined in.

**Selection.** A click on a row selects it; the chevron opens it. Cmd-click (Ctrl-click on Windows and Linux) adds a row to the selection, Shift-click extends it, across levels. Right-clicking offers **Show** and **Copy** for everything selected: Show runs `show(a, b, c, names=["a", "b", "c"])` on the kernel — the variable names become the names in the viewer's tree — with the line appearing in the console; Copy puts the names on the clipboard as `a, b, c`. For a single build123d object it also offers **Export…** — see [Import and export](#import-and-export). `_imported`, the result of Show on a CAD file in the tree, is a plain variable like any other.

**Rows inside a variable.** A row inside a list, tuple, dict or build123d assembly is selected, shown, copied and exported like a variable, through the Python expression that reaches it: `parts[2]`, `parts['lid']`, `hexapod.children[5].children[1]`. What has no index — a shape's topology, a set, a dict entry whose key cannot be written as Python — is reached by its position in iteration, so the Solid of an assembly's first part is `list(hexapod.children[0])[0]`. In the viewer's tree such a row is called by its build123d label, or by its expression where it has none: `show(hexapod.children[0], names=["bottom"])`. A part inside an assembly is shown and exported at its own location, which is relative to its parent assembly: moving the assembly does not move the part shown on its own. Closing a row deselects whatever was selected under it.

### What a click computes

Expanding a build123d shape adds its face, edge and vertex counts. Anything one-dimensional — an `Edge`, a `Wire`, a `Line`, a `Spline` — adds its start and end point (`line @ 0` and `line @ 1` in a script). An edge or a face adds its geometry: line, circle, bspline, plane, cylinder, sphere. The small geometry types open to their values: a `BoundBox` to min, max, size, center and diagonal; a `Vertex` or `Vector` to its coordinates; a `Location` — and so `Pos` and `Rot` — to position and orientation; an `Axis` to position and direction; a `Plane` to its origin and three directions. Each of these is a plain read, under a millisecond whatever the size of the object.

Volume, area and the bounding box of a shape are not computed. On a suspension assembly, each expansion would cost:

| Property                       | Cost per expansion |
| ------------------------------ | ------------------ |
| `faces` / `edges` / `vertices` | 0.8 s together     |
| `area`                         | 2.2 s              |
| `volume`                       | 2.8 s              |
| `bounding_box()`               | 19.1 s             |

Ask for those in the console:

```text
In [2]: rc.volume
```

The viewer shows the extent `bounding_box()` would report, and `show()` prints it.

## Import and export

Both read their options from build123d itself: the dialog shows the parameters of the `import_*` or `export_*` function, with their defaults, so a newer build123d brings its new options with it. Both run their code in the console, where it can be copied into a script, and both remember the options per format for the next time. Cancel at any step ends it.

### Import

**File → Import…** reads a CAD file into a variable.

1. The open dialog asks for the file: BREP, DXF, STEP (`.step`, `.stp`), STL or SVG.
2. A dialog asks for the variable's name — the file's name made into a Python name, `frame-v2.step` becoming `frame_v2` — and shows the importer's options.
3. The import runs on the kernel and shows the result, named after the variable, with the camera fitted to it. The variable appears in the explorer:

```python
from build123d import import_stl, Unit
from build123d_studio import show, Camera
bracket = import_stl("/work/bracket.stl", model_unit=Unit.IN)
show(bracket, names=["bracket"], reset_camera=Camera.RESET)
```

| Format | Importer      | Options                                            |
| ------ | ------------- | -------------------------------------------------- |
| BREP   | `import_brep` | —                                                  |
| DXF    | `import_dxf`  | —                                                  |
| STEP   | `import_step` | —                                                  |
| STL    | `import_stl`  | `model_unit`                                       |
| SVG    | `import_svg`  | `flip_y`, `align`, `ignore_visibility`, `label_by` |

Only options changed from their defaults appear in the call.

**Add import code at cursor position** in the dialog, off by default, also puts the import statement into the open file, where the caret was when Import… was chosen: the assignment, after a `from build123d import …` line for whatever the call needs that the file does not import yet — the importer, and `Unit` or `Align` for an option — where `from build123d import *` covers everything. The `show` stays in the console. The lines go on their own lines: above the caret's line when the caret is at its start, below it otherwise.

### Export

Right-click a variable in the explorer and choose **Export…**. It is offered for a build123d shape, an assembly, or a builder — `BuildPart`, `BuildSketch`, `BuildLine` — whose `part`, `sketch` or `line` is exported, at the top level or [inside a variable](#the-variable-explorer); it is greyed for anything else and when more than one row is selected.

1. A dialog asks for the format — STEP, STL, BREP, OBJ or GLTF — and shows that function's options.
2. **Export** closes the dialog and opens the save panel, which suggests the variable's name with the format's extension — for a row inside a variable its label, or its expression where it has none, with `/`, `:` and the other characters a file name cannot hold replaced by `_`. The name is used as typed.
3. The export runs on the kernel and says what it wrote:

```python
from build123d import export_step, Unit
if export_step(bracket, "/work/bracket.step", unit=Unit.IN):
    print("Exported bracket to /work/bracket.step")
else:
    print("export_step returned False: /work/bracket.step was not written")
```

STEP and GLTF keep an assembly's structure, names and colours. STL and OBJ write one mesh: an assembly is flattened, without names or colours, and the dialog says so.

## Which world the pane describes

The active tab decides:

| Tab                                 | The explorer shows             |
| ----------------------------------- | ------------------------------ |
| **Console**                         | The kernel's namespace         |
| **Run/Debug**, with a debug session | The scopes of the paused frame |
| **Run/Debug**, with nothing running | Nothing — the pane is hidden   |
| **Backend**                         | Nothing — the pane is hidden   |

Nothing switches back on its own: after a run or a session ends, the output stays on screen.
