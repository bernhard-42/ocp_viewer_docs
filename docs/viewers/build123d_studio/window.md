# The window

One window, five surfaces: a folder tree down the left, the editor and the [CAD Viewer](../../viewer.md) across the top, and the console and the variable explorer across the bottom. Everything else — settings, About, the command palette — is a dialog over it.

![](../../assets/build123d-studio.png#only-light)
![](../../assets/build123d-studio-dark.png#only-dark)

The four panes all describe the same object: the script that built it, the model it produced, the console it printed to, and the object unrolled in the explorer.

## The toolbar

Left to right, in four groups. Every button carries its current shortcut in its tooltip, so a rebound chord is right there rather than in this table.

Too narrow a window for all of them and the row scrolls. There is no scrollbar — one drawn across the bottom of a 26-pixel button row covers half of every button, and the platforms disagree about how thin it is allowed to be. **Drag the row sideways with the mouse**, or hold `Shift` and use the wheel. Dragging never presses the button it started on: it becomes a drag only after a few pixels of travel.

| Button                   | What it does                                                                                       |
| ------------------------ | -------------------------------------------------------------------------------------------------- |
| **Toggle the file tree** | Shows or hides the folder sidebar. There is no tree at all until a folder is open.                 |
| **New File**             | A new buffer from the template in **Settings → New file**.                                         |
| **Open File**            | Opens a file in a new tab.                                                                         |
| **Save File**            | Saves the active tab, formatting it first if Format on Save is on.                                 |
| **Run File**             | Saves, then runs the file from disk in a process of its own; pressed again, it stops that process. |
| **Debug File**           | The same with a debugger attached; becomes **Stop Debugging** while a session runs.                |
| **Run Cell**             | Runs the `# %%` cell at the caret on the kernel and moves to the next.                             |
| **Run Selection**        | Runs the selection on the kernel, or the current line if there is no selection.                    |
| **Run All**              | Sends the whole buffer to the kernel.                                                              |
| **Restart kernel**       | Throws the kernel away and starts a new one. The namespace goes with it.                           |
| **Interrupt**            | Interrupts what the kernel is running.                                                             |
| **kernel indicator**     | `starting`, `idle` or `busy` — and it says busy only for code _you_ ran. `busy [+2]` means two more runs are waiting behind it. |
| **health chip**          | Absent while everything is well; see [Health](#health) below.                                      |
| **Settings**             | Packages, new-file template, editor, viewer, debugging, test, shortcuts, application.             |
| **About**                | Versions, the environment path, the log paths, and this instance's Jupyter connection file.        |
| **Command Palette**      | Every editor command by name — see [Commands and shortcuts](commands.md).                          |

## Tabs

The strip above the editor is one tab per open buffer, rebuilt from the buffers themselves. A tab shows a dot while its buffer differs from disk, and two files with the same name are told apart by as much of their path as it takes and no more — `parts/bracket.py` and `spares/bracket.py`, not two identical labels. Overflow scrolls rather than folding into a dropdown.

Choosing a tab puts the keyboard in the editor and restores the caret where it was, so what you type next lands in the file the tab named.

## The folder tree

Present only when a folder is open. It reads a directory the first time that directory is expanded and not before, which is what makes it usable on a CAD project: the folder full of STEP and STL exports is not read until you ask for it. What it has read is kept current by a watcher on the project root, so a file written by a script appears without a refresh — and the header carries **New file**, **New folder** and **Refresh** for the times you want to force it.

Every file is shown, not only Python: a build123d project is exports, images and a README as much as it is `.py`. Three names are hidden because nobody opens them — `__pycache__`, `.git` and `.DS_Store`. Dotfiles in general stay, because `.gitignore` and `.python-version` are files people do open.

Right-clicking a row opens the tree's own menu; renaming a file there moves the tab that holds it with it.

Opening something that is not text is refused rather than attempted — one NUL byte in the first 8 kB is the whole test, the same one git uses — and anything over 10 MB asks first, because the whole buffer goes to the language server on every edit.

## The bottom pane

Three tabs over one pane, and the rule between them is worth knowing because it decides what the variable explorer beside them is describing:

- **Console** — a real `jupyter console` on the kernel your cells run on. See [Console and variables](console_variables.md).
- **Run/Debug** — the output of the separate process started by Run File or Debug File, and while a debug session is live, an evaluate line into the paused frame.
- **Backend** — what the measurement process says as it says it: which shape id was clicked, what it indexed, why a measurement could not be taken. Read-only, and the same lines go to `backend.log`.

**The tab decides what the panes beside it are about.** Console means the kernel, so the explorer shows the kernel's namespace. Run/Debug with a session behind it means the debugged process, so the explorer shows the paused frame. Run/Debug with nothing behind it — a plain Run File, a finished session, or simply having clicked the tab — means nothing, so the explorer goes away rather than showing a namespace the output is not about. Nothing switches back on its own, because the output is still there and still worth reading.

While a debug session is live, the step controls — continue, step over, step into, step out, restart, stop — appear in that same tab row, beside the output they act on.

## Health

Everything below the window has a lifecycle of its own — the model channel that carries `show()`, the measurement process, the kernel, the language server, the console — and each can die while the window still looks entirely well. The chip in the toolbar is what says so.

It is hidden while everything is well, because a status light that is always on is a status light nobody reads. It appears as `degraded` (working, with something worth knowing) or `failed` (not working), showing the worst state anything is in; its tooltip lists every subsystem with what it last said. Clicking it opens the **Backend** tab, which is where the reason is. A subsystem that newly fails brings that tab forward on its own — once, not every time it speaks again.

`starting` is not a fault and is not shown: at the moment the window appears, the measurement process is still loading the geometry kernel and will be for a second or two.

## Layout

The two rows split independently — the editor/viewer ratio and the console/explorer ratio are separate numbers — because a single grid would force one to follow the other. Drag any splitter; the result is remembered, as is the window's own size and position, in the [data directory](first_run.md#where-it-all-lives) rather than beside the application, so an update does not take the layout with it.

**View → Toggle Sidebar** hides the tree without closing the folder, and **View → Toggle Console and Variables** hides the whole bottom row. Both survive a restart. Which files were open, and where the caret was in each, is restored too.

## Theme

Untouched, the whole window follows the desktop's own light or dark setting and changes with it. **Settings → Application → Appearance → Dark mode** pins it either way, and it then stops following. It is one setting for the window rather than one per surface — the editor, the viewer and the terminal all move together.
