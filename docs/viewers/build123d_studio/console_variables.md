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

**Selection.** A click on a row selects it; the chevron opens it. Cmd-click (Ctrl-click on Windows and Linux) adds a row to the selection, Shift-click extends it. Right-clicking offers **Show** and **Copy** for everything selected: Show runs `show(a, b, c)` on the kernel, the line appearing in the console; Copy puts the names on the clipboard as `a, b, c`. Rows below a variable are addressed by position and have no name, so they cannot be selected. `_imported`, the result of Show on a CAD file in the tree, is a plain variable like any other.

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

## Which world the pane describes

The active tab decides:

| Tab                                 | The explorer shows             |
| ----------------------------------- | ------------------------------ |
| **Console**                         | The kernel's namespace         |
| **Run/Debug**, with a debug session | The scopes of the paused frame |
| **Run/Debug**, with nothing running | Nothing — the pane is hidden   |
| **Backend**                         | Nothing — the pane is hidden   |

Nothing switches back on its own: after a run or a session ends, the output stays on screen.
