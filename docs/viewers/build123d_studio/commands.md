# Commands and shortcuts

## The keymap

These eighteen commands are the ones Studio owns and the ones the [Shortcuts tab](#rebinding) edits. Everything else in the editor — find, replace, comment toggling, folding, formatting — is Monaco's and lives in the [command palette](editor.md#the-command-palette).

| Command                     | macOS          | Windows and Linux        |
| --------------------------- | -------------- | ------------------------ |
| Run Cell                    | `⇧↩`           | `Shift + Enter`          |
| Run Cell (Keep Cursor)      | `⌃↩` or `⌘↩`   | `Ctrl + Enter`           |
| Run Selection or Line       | `⌃⇧↩` or `⌘⇧↩` | `Ctrl + Shift + Enter`   |
| Run All                     | `⌥↩`           | `Alt + Enter`            |
| Run Cell Above              | —              | —                        |
| Run All Above               | —              | —                        |
| Run All Below               | —              | —                        |
| Run File                    | `⌃F5`          | `Ctrl + F5`              |
| Restart Kernel              | `⇧⌥⌘R`         | `Ctrl + Shift + Alt + R` |
| Test File                   | —              | —                        |
| Test Folder                 | —              | —                        |
| Debug File _(and Continue)_ | `F5`           | `F5`                     |
| Continue                    | —              | —                        |
| Step Over                   | `F10`          | `F10`                    |
| Step Into                   | `F11`          | `F11`                    |
| Step Out                    | `⇧F11`         | `Shift + F11`            |
| Restart Debugging           | `⌃⇧F5`         | `Ctrl + Shift + F5`      |
| Stop Debugging              | `⇧F5`          | `Shift + F5`             |

Three groups of decisions are behind that table, and they are all borrowed rather than invented:

- **The cell chords are Jupyter's.** `Shift-Enter` runs and advances, `Ctrl-Enter` runs and stays — including `Cmd-Enter` on macOS, which Jupyter also accepts there. That pair is the muscle memory most build123d users already have, so it is matched exactly rather than improved on. Jupyter has no "run line", and the chord left over takes it.
- **The debug chords are VS Code's**, `F5` included: it starts a session and continues a paused one, which is why Continue has no chord of its own. `Ctrl-F5` for Run File is VS Code's Run Without Debugging, and it is literally Control on macOS too, as it is there.
- **The two Test commands have no chord either.** Each asks where to look before it does anything, which makes it a deliberate act rather than one worth repeating from the keyboard — the Shortcuts tab will bind them for anybody who runs the suite all day.
- **The three marker commands have no chord.** They are about a `# %%` marker rather than about the caret, which is what makes them worth a click and awkward as a keystroke. They are the buttons above each marker, they are in the Run menu, and the Shortcuts tab will bind them for anybody who disagrees.

Restart Kernel is delivered whatever has the keyboard. The rest are editor actions, so they arrive when the editor has focus — which is the difference you want: a restart is what you reach for while the console is wedged or the explorer has the caret.

Every toolbar button's tooltip carries its command's _current_ chord, so a rebinding is visible where the button is rather than only in Settings.

## Rebinding

**Settings → Shortcuts** puts a dialog over the same map.

- Each command shows its chords as chips. **Click a chip to remove that chord.**
- **Record** captures the next chord you press. `Escape` cancels.
- A command may answer to several chords; add as many as you like.
- **Restore all defaults** puts the table above back.

Nothing is written until **Apply**, so recording a chord is not a change you cannot cancel. Once applied, the editor is re-bound straight away and the menus follow — one map, read by both, is the only arrangement in which they cannot drift apart.

Two things are checked. A chord with no modifier is refused unless it is a function key, because it would swallow that key while typing. A chord bound to two commands is _reported_ rather than refused — which of the two you meant to keep is yours to decide — with a line saying whichever registers last wins.

## The menus

On macOS this is the system menu bar. On Windows and Linux it is drawn in the window's own title bar; `Alt` shows the underlines and `Alt-F` opens File, and the arrow keys walk it from there.

| Menu                           | Items                                                                                                          |
| ------------------------------ | -------------------------------------------------------------------------------------------------------------- |
| **build123d Studio** _(macOS)_ | About, Settings…, Quit                                                                                         |
| **File**                       | New, Open File…, Open Folder… · Save, Save As…, Save All · Close, Close All, Close Folder · Exit _(off macOS)_ |
| **View**                       | Toggle Sidebar, Toggle Console and Variables                                                                   |
| **Edit**                       | Cut, Copy, Paste                                                                                               |
| **Run**                        | The four cell commands · the three marker commands · Restart Kernel · Run File · the seven debug commands      |
| **Test**                       | Test File, Test Folder                                                                                         |
| **Help** _(off macOS)_         | Settings…, About                                                                                               |

The Edit menu is not decoration. Without a menu, macOS binds the standard editing shortcuts to nothing at all and `Cmd-C` in the console or in a dialog simply does nothing — the keystroke is swallowed before the page sees it. The menu is the mechanism that delivers it, and on macOS the items carry the platform's own roles, so Cut, Copy and Paste land on whatever actually has focus.

The separators in the Run menu carry the meaning. Everything above Restart Kernel runs *on the kernel* and leaves its names in the namespace the console shares; everything below it runs in a process of its own and leaves nothing behind. Testing is a menu rather than two more Run items because nothing about it starts from the buffer on screen — see [Tests](running.md#tests).

Items that need something to act on are greyed rather than hidden: Close needs a tab, Close Folder needs a project, and the step commands need a session — so the Run menu keeps one shape and you can see what debugging will offer before you start it.

!!! note "Accelerators in the menu"

    The File and View items show `⌘N`, `⌘S`, `⌘B`, `⇧⌘Y` and so on. Only macOS binds them, and only as Command plus a single character — that is the limit of what the window toolkit's menu offers. So `⇧⌘Y` appears beside its label rather than in the right-aligned column, and on Windows and Linux these read as documentation: use the toolbar button, or reach the item through `Alt`. The commands in the table at the top of this page are bound by the editor on every platform and are not affected.
