# Debugging

**Debug File** is [Run File](running.md#as-a-file-in-a-process-of-its-own) with a debugger attached: the buffer is saved and the file on disk runs in a process of its own, under [debugpy](https://github.com/microsoft/debugpy). None of it touches the kernel — the console keeps working throughout, the namespace is the same afterwards as it was before, and the objects the debugged file made die with it.

## Breakpoints

Click in the gutter beside a line number to set one, click it again to remove it.

They are deliberately not persisted. A breakpoint is something you put down for the next run and forget, and a file that reopens with last week's breakpoints in it stops somewhere nobody asked for.

## Starting and stepping

| Command               | Default chord   |
| --------------------- | --------------- |
| **Debug File**        | `F5`            |
| **Continue**          | `F5`            |
| **Step Over**         | `F10`           |
| **Step Into**         | `F11`           |
| **Step Out**          | `Shift-F11`     |
| **Restart Debugging** | `Ctrl-Shift-F5` |
| **Stop Debugging**    | `Shift-F5`      |

These are VS Code's, deliberately. `F5` covers both start and resume — it continues a paused session rather than refusing — which is why Continue has no chord of its own.

The same six commands appear as buttons in the bottom pane's tab row while a session is live, beside the output they act on. The line execution is stopped at is highlighted in the editor.

## What the panes show

While a session runs, the **Run/Debug** tab is the debugged process: its output, and an evaluate line at the foot of it that runs an expression in the frame execution stopped in. The variable explorer beside it shows that frame's scopes, `Locals` first, expandable to whatever depth the data has.

The **Console** tab still describes the kernel. That is the rule the whole layout follows — the tab decides what the panes beside it are about, so there is never a question of which world you are looking at. When the session ends, the Run/Debug tab keeps its output and the explorer beside it empties, because the frames it was describing are gone with the process.

The viewer is the exception, as it is for a plain run: a `show()` from the debugged process reaches the viewer already on screen, because the process is given the kernel's viewer address.

## Visual debugging

**Settings → Debugging** carries an expression that runs every time execution stops. Its default is

```python
from build123d_studio import show_all; show_all(locals())
```

so stepping through a model is watching it being built. The import is part of the default rather than an assumption about your file: the expression is evaluated _in the paused frame_, and a file that draws with `show()` — or that has not drawn anything yet — has never bound `show_all`. Re-importing at every stop costs nothing after the first.

It costs a tessellation per stop, which on a large assembly is seconds. Two ways out: **Only at breakpoints, not at every step** limits it to the places you deliberately stopped, and clearing the field switches it off altogether. Out of the box it runs at every stop, which is what somebody starting a debugger expects to see.

If the expression fails, the failure is written to the debug console — an expression that raises on every step needs saying once, in a place you are already looking.

## Step into my code only

On by default, which is debugpy's own default: **Step Into** stays inside your own file. Turned off, it follows the call into build123d — which is what somebody debugging a model wants once they know to want it, and disorienting before that.

!!! note

    A step through a multi-line call stops more often than it looks like it should: `base -= Cylinder(a, b, c)` was measured stopping six times. That is Python's line events, not the debugger being confused — and it is the reason the on-stop expression has a "breakpoints only" setting at all.
