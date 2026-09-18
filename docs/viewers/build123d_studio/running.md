# Running code

There are two ways to run something, and they behave differently.

## On the kernel

**Run Cell**, **Run Selection** and **Run All** send code to a Jupyter kernel that stays alive between runs; the [Console](console_variables.md) at the bottom is a `jupyter console` on that same kernel. Everything you run leaves its names behind: build a shape with `Shift-Enter`, then type `part.volume` in the console and get an answer. The [variable explorer](console_variables.md#the-variable-explorer) shows the same namespace.

This is the mode for building something up incrementally.

### Cells

`# %%` on a line of its own marks a cell boundary, as in Jupyter, VS Code, Spyder and jupytext. Everything up to the next marker belongs to that cell; text before the first marker is a cell of its own; a file with no markers is one cell.

| Command                    | Default chord      | What it sends                                       |
| -------------------------- | ------------------ | --------------------------------------------------- |
| **Run Cell**               | `Shift-Enter`      | The cell at the caret, then moves to the next       |
| **Run Cell (Keep Cursor)** | `Ctrl-Enter`       | The cell at the caret, staying where you are        |
| **Run Selection or Line**  | `Ctrl-Shift-Enter` | The selection, or the current line if there is none |
| **Run All**                | `Alt-Enter`        | The whole buffer                                    |
| **Run Cell Above**         | —                  | The cell before this one                            |
| **Run All Above**          | —                  | Everything up to this marker                        |
| **Run All Below**          | —                  | This marker and everything after it                 |

The last three have no chord by default. They are buttons above every `# %%` marker — five, one of them an interrupt — on by default and switched off in **Settings → Editor → Show cell actions**; they are in the **Run** menu; and the [Shortcuts tab](commands.md#rebinding) can bind them.

The cell chords are Jupyter's, including `Cmd-Enter` on macOS.

### Restart and interrupt

**Interrupt** interrupts what the kernel is running; with nothing running it does nothing. Python raises `KeyboardInterrupt` between operations, so a single long native call — a boolean on a large assembly — cannot be interrupted; five seconds after an interrupt that has not been obeyed, Studio says so and offers a restart.

**Restart kernel** starts a new kernel; every name in the namespace goes with the old one. Its chord is four keys (`Shift-Alt-Cmd-R`, `Ctrl-Shift-Alt-R` elsewhere) and works whatever has the keyboard, unlike the run commands, which need the editor to have focus.

### Running while something is running

The kernel takes one request at a time; a Run pressed while a cell is running is **queued**, as in Jupyter. The kernel indicator shows how many are waiting — `busy`, `busy [+1]`, `busy [+2]` — and counts down as they run. A queued run appears in the console when the kernel starts it, with its `In [n]`.

After any run the keyboard returns to the editor, so the caret is visible where the run left it.

## As a file, in a process of its own

**Run File** (`Ctrl-F5`) and **Debug File** (`F5`) save the buffer and run the _file from disk_ in a separate process with the same interpreter and environment. Nothing it defines reaches the kernel; when it ends, the process and everything in it is gone.

This is the mode for "does this script work from a clean start" — the check that finds an import the kernel's accumulated namespace was hiding.

Output appears in the **Run/Debug** tab, never in the Console tab: the Console tab is the kernel, the Run/Debug tab is the other process. Pressing Run File again while it runs stops it, as does the **Stop** button that appears in the bottom pane's tab row. A run cannot start while a debug session is live.

A `show()` in the file reaches the viewer you are looking at: the process is given the viewer's address. Import it as `from build123d_studio import show`, or with the universal `from ocp_viewer_core.viewer import show`, which finds this viewer through the environment the process was started with. `from ocp_vscode import show` does not reach it — that is a client for the VS Code extension's viewer.

### The PATH a run gets

Everything Studio starts — the kernel, the console, Run File, tests, make — sees the environment's own `bin` (`Scripts` on Windows) first on `PATH`, so `python`, `pytest` and `ruff` are the ones Studio installed. On macOS and Linux, what your login shell puts on `PATH` follows: Studio asks your account's default shell once at startup, the way an editor launched from the dock does, so Homebrew and the like are there even though the application was not started from a terminal. Windows keeps the `PATH` the system gives every application. The Backend tab's log has a `PATH:` line saying what was adopted.

Debugging is the same mechanism with a debugger attached — see [Debugging](debugging.md).

## Tests

**Test → Test File** and **Test → Test Folder** run `pytest` over what you pick. Each opens a chooser — a `.py` file, or a folder — and runs `python -m pytest <what you chose>` the way Run File runs a file: the report arrives in the **Run/Debug** tab, Stop ends the run, and the process is gone when it finishes.

`pytest` is part of the environment, like `ruff` and the language server; you do not install it.

Both items save every unsaved buffer before they start — pytest reads from disk.

One child process runs at a time. Asking for a test run while something else is running says so, and where Stop is.

!!! note "It is `pytest`, not a test explorer"

    There is no tree of tests, no green ticks and no re-run-failed. pytest prints its report, and the report is the feature.

**Settings → Test** has one switch. **Ignore warnings** adds `-W ignore` to the run. It is off by default, pytest's own behaviour; turn it on when a hundred tests each raise the same `DeprecationWarning` and bury the summary.

## Make

A project with a `Makefile` — `Makefile`, `makefile` or `GNUmakefile` — can run its targets from the tree: right-click the file and its targets are listed below a line, **Make ▸ build**, **Make ▸ test**, … Picking one runs `make <target>` in the Makefile's folder the way a test run runs: the output arrives in the **Run/Debug** tab, **Stop** ends it, and one such process runs at a time.

The list is read from the file at every right-click, so a target added in a terminal is in the next menu. It is the rules written in the file, in their order, without make's own dot targets; a target that expects a variable — `bump part=minor` — is listed too and runs without it, and make says what it wanted. The entries appear only when `make` is on the `PATH` a run gets; without it, the Makefile is a file like any other.

On Windows, GNU make runs recipes through `sh.exe` if one is on `PATH` and through `cmd.exe` otherwise, exactly as it would from a terminal; a Makefile written for a Unix shell fails there in the same way it would in a command prompt.

## Drawing

```python
from build123d import *
from build123d_studio import show

b = Box(1, 2, 3)
show(b)
```

`show_all()` draws everything drawable in scope. New files start from a template with the imports in it, editable in **Settings → New file**.

Everything the shared API offers — [`show`](../../show.md), [`show_object`](../../show_object.md), [`push_object`](../../push_object.md), [`show_all`](../../show_all.md), [`set_defaults`](../../config.md), the [colour maps](../../colormaps.md), [materials](../../pbr_studio.md) and [animation](../../animation.md) — is exported by `build123d_studio` under the names the other viewers use. There is one viewer per window, so there is no `port` keyword.

## Where your code runs

With a folder open, the kernel's working directory is the folder root, whichever file you are editing, so a relative path means the same thing everywhere in the project. With no folder open, the working directory follows the active file.

One folder is open at a time. Opening another closes the tabs of the old one, asking once about anything unsaved.
