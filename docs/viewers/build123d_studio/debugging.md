# Debugging

**Debug File** is [Run File](running.md#as-a-file-in-a-process-of-its-own) with a debugger attached: the buffer is saved and the file on disk runs in a process of its own, under [debugpy](https://github.com/microsoft/debugpy). The kernel is not involved: the console keeps working, the namespace is unchanged afterwards, and the objects the debugged file made die with its process.

## Breakpoints

Click in the gutter beside a line number to set one, click again to remove it. Breakpoints are not persisted across sessions.

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

These are VS Code's chords. `F5` starts a session and continues a paused one, so Continue has no chord of its own.

The same six commands are buttons in the bottom pane's tab row while a session is live. The line execution is stopped at is highlighted in the editor.

## What the panes show

While a session runs, the **Run/Debug** tab is the debugged process: its output, and an evaluate line at the foot that runs an expression in the frame execution stopped in. The variable explorer beside it shows that frame's scopes, `Locals` first, expandable to any depth.

The **Console** tab still shows the kernel. When the session ends, the Run/Debug tab keeps its output and the explorer beside it empties: the frames are gone with the process.

A `show()` from the debugged process reaches the viewer on screen, as from a plain run: the process is given the viewer's address.

## Visual debugging

**Settings → Debugging** holds an expression that runs every time execution stops. Its default is

```python
from build123d_studio import show_all; show_all(locals())
```

so stepping through a model is watching it being built. The expression is evaluated in the paused frame, which is why the import is part of it: a file that draws with `show()`, or has not drawn yet, has not bound `show_all`. The import costs nothing after the first time.

Each stop costs a tessellation, which on a large assembly is seconds. **Only at breakpoints, not at every step** limits it to the places you set; clearing the field switches it off.

If the expression fails, the failure is written to the debug console.

## Step into my code only

On by default: **Step Into** stays inside your own file. Turned off, it follows the call into build123d.

!!! note

    A step through a multi-line call stops more often than it looks like it should: `base -= Cylinder(a, b, c)` stops six times. Those are Python's line events. This is what the "breakpoints only" setting above is for.
