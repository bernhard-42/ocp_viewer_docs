# The editor

The editor is [Monaco](https://microsoft.github.io/monaco-editor/), the editor VS Code is built on. Python language support comes from the application's own environment.

## Completion, hover and signatures

Two sources answer, merged in the sidecar before anything reaches the editor.

**The language server** ([basedpyright](https://github.com/DetachHead/basedpyright), running in the same environment your code runs in) reads the buffer without executing it. It resolves `from build123d import *` to `Box`, follows `b = Box(...)` to `b.center`, and knows a function's keyword arguments before the function has run.

**The kernel** answers from the live namespace — IPython's own Tab completion. It knows the methods of the part you built two cells ago, which exists only in the kernel that ran it.

Where both have a name, the kernel's entry wins, because it carries the signature IPython worked out.

The same connection answers **hover** — what a name holds; build123d operations return `Part`, `Sketch`, `Curve` and `Compound` almost interchangeably — and **parameter hints** for the call the caret is inside, with the argument you are typing highlighted.

## Squiggles

The language server reports what it finds wrong with the buffer and the editor underlines it, refreshing as you type.

## Formatting

The formatter is [ruff](https://docs.astral.sh/ruff/), run in the sidecar.

- `Shift-Alt-F`, or **Format Document** from the palette or the context menu, formats the buffer.
- **Format on save** is on by default. ruff changes layout, never meaning; the result is one undo away; a buffer that does not parse is saved as it is.
- **Line length** in **Settings → Editor** is both the width ruff wraps at and the column of the editor's ruler. The default is 88, ruff's own.

ruff runs isolated: the line length in Settings is the one used, whatever a project's `pyproject.toml` or `ruff.toml` says.

Above **50 000 lines** a buffer is saved without formatting, and the log says so.

## Find

`Cmd`/`Ctrl-F` opens Monaco's find widget. `Alt-Enter` there selects **every** match, as in VS Code, so the next thing typed replaces all of them. Elsewhere in the editor `Alt-Enter` is **Run All**.

## Snippets

Type a prefix and accept the suggestion. The shipped set is the build123d CodeCAD speedmodeling snippets by Jern, from [build123d-portable](https://github.com/build123d/build123d-portable), with the viewer import pointed at `build123d_studio`.

Prefixes begin with a punctuation mark, so they never collide with an identifier the language server suggests, and the mark names the style: `?` is builder mode, `>` is algebra mode.

| Prefix      | Inserts                                                    |
| ----------- | ---------------------------------------------------------- |
| `?template` | The file template — imports, `show_clear()`, a `# %%` cell |
| `?bdp`      | `with BuildPart() as p:`                                   |
| `?bds`      | `with BuildSketch() as s:`                                 |
| `?ebds`     | A `BuildSketch` block with an `extrude` under it           |
| `?bdl`      | `with BuildLine() as l:`                                   |
| `?loc`      | `with Locations(()):` — and `?ploc`, `?gloc`, `?hloc`      |
| `?ext`      | `extrude(amount=)` — and `?off`, `?spl`, `?mirr`           |
| `>ext`      | `extrude(obj, amount=)`, the algebra-mode form             |
| `>cha`      | `chamfer(obj, )` — and `>spl`, `>mirr`, `>loc`, `>part`    |

Twenty-five in all; typing `?` or `>` alone narrows the list to one family.

The snippets are in `snippets.json` beside the settings, in the [data directory](first_run.md#where-it-all-lives). Studio writes it from the shipped set the first time it starts on a machine; from then on the file is the whole set, read at every start and when Settings is applied, so an edit, an addition or a removal takes effect at the next start. It is VS Code's `.code-snippets` shape, read with the same parser VS Code uses: comments and trailing commas are fine, and a `.code-snippets` file pasted in unedited works. Delete the file to get the shipped set back.

## The new-file template

**Settings → New file** holds what a new buffer starts with. Cleared, **New File** opens an empty buffer; **Restore default** puts the shipped template back.

It is a snippet, in the same syntax: `$1` and `${1:like this}` are stops to tab between, `${1|Box,Cylinder,Sphere|}` offers a choice, `$0` is where the caret ends, and `$CURRENT_YEAR`, `$RANDOM` and `$CLIPBOARD` are filled in. A literal dollar is `\$`.

## The command palette

`F1` opens Monaco's palette, which lists every editor command by name — find, replace, comment toggling, folding, formatting, and the run and debug commands. It is Monaco's, not the [keymap](commands.md), so it is not editable in Settings, and it works while the caret is in the editor. The toolbar button works from anywhere and focuses the editor first.

## Saving

A save writes **into** the file: hard links still see the new contents, and extended attributes — Finder tags on macOS — ownership and permissions survive. A symlink is followed; the real file receives the edits and the symlink stays.

If the file changed on disk — a formatter, a `git checkout`, a second window — you are asked before anything is overwritten. The question comes when you **focus the window or choose the tab**, not only on save. **Reload** takes what is on disk, **Overwrite** writes your buffer over it, and **Cancel** marks the tab modified so the choice survives being closed.

Unsaved buffers are copied into the data directory as you type; the copy goes when it is no longer the only one — a save, a closed tab, a normal quit. Whatever is left at the next start is offered back. The copies are plain source files and can be opened in any editor while the application is not running. A buffer above 50 000 lines is copied once, immediately before each save, instead of continuously.

## Opening files

Every file in the tree can be opened, not only Python. What the editor does for a file depends on its extension:

| Language | Highlighting | Diagnostics | Folding | Completion | Special |
| --- | --- | --- | --- | --- | --- |
| Python `.py` `.pyi` | yes | basedpyright | yes | basedpyright + kernel | ruff on save |
| JSON `.json` | yes | syntax | yes | — | JSONC accepted |
| YAML `.yaml` `.yml` | yes | — | indentation | — | |
| TOML `.toml` | yes | — | indentation | — | |
| other | plain text | — | — | — | |

Only Python goes to the language server and the formatter. JSON is checked by Monaco's JSON service, which accepts comments and trailing commas (JSONC) — `snippets.json` is written that way. TOML is highlighting only. Binary files are not opened: a file with a NUL byte in its first 8 kB is refused (git's own test for text). Files over 10 MB ask before opening: Monaco itself handles large files well, but a Python buffer is sent whole to the language server on open and after every edit.
