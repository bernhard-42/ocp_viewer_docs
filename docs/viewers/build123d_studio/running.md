# Running code

There are two ways to run something, and they behave differently on purpose. Understanding the difference is worth more than anything else on this page.

## On the kernel

**Run Cell**, **Run Selection** and **Run All** send code to a Jupyter kernel that stays alive between runs, and the [Console](console_variables.md) at the bottom is a real `jupyter console` attached to that same kernel. So everything you run leaves its names behind: build a shape with `Shift-Enter`, then type `part.volume` in the console below and get an answer. The [variable explorer](console_variables.md#the-variable-explorer) shows that same namespace and expands on demand.

This is the mode for building something up incrementally, and it is where the application's Jupyter heritage shows.

### Cells

`# %%` on a line of its own marks a cell boundary, as in Jupyter, VS Code, Spyder and jupytext. Everything up to the next marker belongs to that cell, text ahead of the first marker is a cell in its own right, and a file with no markers at all is one cell — so "run cell" means something in a plain script too.

| Command                    | Default chord      | What it sends                                       |
| -------------------------- | ------------------ | --------------------------------------------------- |
| **Run Cell**               | `Shift-Enter`      | The cell at the caret, then moves to the next       |
| **Run Cell (Keep Cursor)** | `Ctrl-Enter`       | The cell at the caret, staying where you are        |
| **Run Selection or Line**  | `Ctrl-Shift-Enter` | The selection, or the current line if there is none |
| **Run All**                | `Alt-Enter`        | The whole buffer                                    |
| **Run Cell Above**         | —                  | The cell before this one                            |
| **Run All Above**          | —                  | Everything up to this marker                        |
| **Run All Below**          | —                  | This marker and everything after it                 |

The last three have no chord by default: they are about a _marker_ rather than about the caret, which is what makes them worth a click and awkward as a keystroke. They appear instead as buttons above every `# %%` marker — five of them, one of which is an interrupt — which is on by default and switched off in **Settings → Editor → Show cell actions**. They are in the **Run** menu as well, and the [Shortcuts tab](commands.md#rebinding) will bind them for anybody who disagrees.

The two cell chords are Jupyter's, deliberately — including `Cmd-Enter` on macOS, which Jupyter also accepts there.

### Restart and interrupt

**Interrupt** interrupts what the kernel is running. **Restart kernel** throws the kernel away and starts a new one, taking every name in the namespace with it; it is four keys (`Shift-Alt-Cmd-R`, `Ctrl-Shift-Alt-R` elsewhere) because it is not a thing to hit by accident. That chord is delivered whatever has the keyboard, unlike the run commands, which are editor actions — a restart is what you reach for while the console is wedged or the explorer has the caret.

## As a file, in a process of its own

**Run File** (`Ctrl-F5`) and **Debug File** (`F5`) save the buffer and run the _file from disk_ in a separate process, with the same interpreter and the same environment. Nothing it defines reaches the kernel, and when it ends the process is gone with everything in it.

That is the mode for "does this script actually work from a clean start", and it is what you want when the kernel's accumulated namespace is hiding a missing import.

Output appears in the **Run/Debug** tab beside the console, never in the Console tab — the Console tab is the kernel and the Run/Debug tab is the other process, and neither ever describes the other. Pressing Run File again while it runs stops it. Starting a run while a debug session is live is refused rather than allowed to produce two processes claiming the same panes.

The viewer is the deliberate exception: a `show()` in the file reaches the same viewer you were already looking at, because the process is given the viewer's address. A picture is not state.

Debugging is the same mechanism with a debugger attached — see [Debugging](debugging.md).

## Drawing

```python
from build123d import *
from build123d_studio import show

b = Box(1, 2, 3)
show(b)
```

`show_all()` draws everything drawable in scope, which is usually what you want while exploring. New files start from a template that already has the imports in it, and the template is editable in **Settings → New file**.

Everything the shared API offers — [`show`](../../show.md), [`show_object`](../../show_object.md), [`push_object`](../../push_object.md), [`show_all`](../../show_all.md), [`set_defaults`](../../config.md), the [colour maps](../../colormaps.md), [materials](../../pbr_studio.md) and [animation](../../animation.md) — is exported by `build123d_studio` under exactly the names the other viewers use. There is one viewer per window, so no `port` keyword exists and none is needed.

## Where your code runs

With a folder open, the kernel's working directory is the folder root and stays there, whichever file you are editing — so a relative path in your code means the same thing everywhere in the project. With no folder open, the working directory follows the active file instead.

One folder is open at a time. Opening another closes the tabs from the old one, asking once about anything unsaved.
