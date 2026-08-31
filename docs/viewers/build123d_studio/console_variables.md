# Console and variables

The bottom row of the window is the kernel described twice: as a transcript you can type at, and as a table you can open.

## The console

The **Console** tab is a real `jupyter console` attached to the same kernel [Run Cell](running.md#on-the-kernel) sends to. Nothing is interpreted on the way: keystrokes go to a pty, bytes come back and are written verbatim. So it behaves like IPython because it _is_ IPython — line editing, history search, tab completion, `?` and `??`, and the magics all come from the other side rather than from a widget imitating them.

That is what makes the two halves of the window fit together: run a cell with `Shift-Enter`, then ask the console about what it left behind.

```text
In [1]: part.volume
Out[1]: 1256.6370614359173
```

Ten thousand lines of scrollback are kept, so a traceback from earlier in the session is still there. Right-clicking offers copy and paste; on macOS the standard `Cmd-C` / `Cmd-V` work through the application menu, which is what gets them delivered into a pane the browser would otherwise swallow them in.

The kernel's connection file is named in **Help → About**, for attaching anything else to the same kernel.

## The variable explorer

The pane beside the console lists what is in the namespace: name, type, and a short `repr`, with a length in brackets for anything sized and build123d's own `label` where an object carries one. Drag the boundaries between the columns to re-proportion them; the widths are remembered.

Rows are pushed by the sidecar whenever the kernel goes idle, so the table is current after a run and after typing in the console, with nothing to refresh and no polling. That listing is deliberately cheap — it reads only what is already there, never anything that computes geometry.

Expanding a row asks for its contents at that moment, and goes as deep as the data does: a row inside a row inside a row is addressed by position, so an assembly unrolls by its children the way it is built. Long collections are paged rather than truncated.

### What a click will and will not compute

Expanding a build123d shape adds its face, edge and vertex counts. It does not add volume, area or the bounding box, and that is measured rather than cautious — on a suspension assembly, with OCCT caching none of it:

| Property                       | Cost per expansion |
| ------------------------------ | ------------------ |
| `faces` / `edges` / `vertices` | 0.8 s together     |
| `area`                         | 2.2 s              |
| `volume`                       | 2.8 s              |
| `bounding_box()`               | 19.1 s             |

The counts are enough to tell two versions of a model apart, which is what the pane is for. The rest belongs in the console, where you ask for it and can see what it costs:

```text
In [2]: rc.volume
```

The viewer already shows the extent `bounding_box()` would report, and `show()` prints it.

## Which world the pane is describing

The tab on show decides, and only that:

| Tab on show                         | The explorer shows             |
| ----------------------------------- | ------------------------------ |
| **Console**                         | The kernel's namespace         |
| **Run/Debug**, with a debug session | The scopes of the paused frame |
| **Run/Debug**, with nothing running | Nothing — the pane goes away   |
| **Backend**                         | Nothing — the pane goes away   |

The two worlds share no addresses, so nothing is carried across when it switches. Nothing switches back on its own either: after a run or a session ends, the output is still there and still worth reading.
