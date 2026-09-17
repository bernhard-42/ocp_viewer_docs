# Commands and shortcuts

## The keymap

These eighteen commands are Studio's own; the [Shortcuts tab](#rebinding) edits them. Everything else in the editor — find, replace, comment toggling, folding, formatting — is Monaco's and is in the [command palette](editor.md#the-command-palette).

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

The cell chords are Jupyter's: `Shift-Enter` runs and advances, `Ctrl-Enter` (also `Cmd-Enter` on macOS) runs and stays. The debug chords are VS Code's: `F5` starts a session and continues a paused one, so Continue has no chord of its own; `Ctrl-F5` is Run File on every platform, Control on macOS too. The Test commands and the three marker commands have no chord by default — they are in the menus and above each `# %%` marker — and can be bound in the Shortcuts tab.

Restart Kernel works whatever has the keyboard. The other commands are editor actions and need the editor to have focus.

Every toolbar button's tooltip shows its command's current chord.

## Rebinding

**Settings → Shortcuts** edits the same map.

- Each command shows its chords as chips. **Click a chip to remove that chord.**
- **Record** captures the next chord you press. `Escape` cancels.
- A command may have several chords.
- **Restore all defaults** restores the table above.

Nothing is written until **Apply**. After Apply the editor and the menus use the new map immediately.

A chord with no modifier is refused unless it is a function key. A chord bound to two commands is reported, not refused: the command that registers last gets it.

## The menus

On macOS the menu is the system menu bar. On Windows and Linux it is in the window's title bar: `Alt` shows the underlines, `Alt-F` opens File, the arrow keys walk the menus.

| Menu                           | Items                                                                                                          |
| ------------------------------ | -------------------------------------------------------------------------------------------------------------- |
| **build123d Studio** _(macOS)_ | About, Settings…, Quit                                                                                         |
| **File**                       | New, Open File…, Open Folder… · Save, Save As…, Save All · Close, Close All, Close Folder · Exit _(off macOS)_ |
| **View**                       | Toggle Sidebar, Toggle Console and Variables                                                                   |
| **Edit**                       | Cut, Copy, Paste                                                                                               |
| **Run**                        | The four cell commands · the three marker commands · Restart Kernel · Run File · the seven debug commands      |
| **Test**                       | Test File, Test Folder                                                                                         |
| **Help** _(off macOS)_         | Settings…, About                                                                                               |

On macOS the Edit menu is what delivers `Cmd-C`, `Cmd-X` and `Cmd-V` to the console and to dialogs; its items use the platform's own roles, so they act on whatever has focus.

In the Run menu, everything above Restart Kernel runs on the kernel and leaves its names in the namespace the console shares; everything below runs in a process of its own. Testing has its own menu — see [Tests](running.md#tests).

Items that need something to act on are greyed, not hidden: Close needs a tab, Close Folder a project, the step commands a session.

!!! note "Accelerators in the menu"

    The File and View items show `⌘N`, `⌘S`, `⌘B`, `⇧⌘Y` and so on. Only macOS binds them, and only as Command plus a single character, so `⇧⌘Y` appears beside its label rather than in the right-aligned column. On Windows and Linux they are documentation: use the toolbar button or reach the item through `Alt`. The commands in the table at the top of this page are bound by the editor on every platform.
