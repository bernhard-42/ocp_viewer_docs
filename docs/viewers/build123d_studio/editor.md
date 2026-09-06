# The editor

The editor is [Monaco](https://microsoft.github.io/monaco-editor/), the one VS Code is built on, with the Python language support supplied by the application's own environment rather than by the browser.

## Completion, hover and signatures

Two sources answer, and they are merged in the sidecar before anything reaches the editor.

**The language server** ([basedpyright](https://github.com/DetachHead/basedpyright), running in the same environment your code runs in) reads the buffer and infers without executing it. It resolves `from build123d import *` to `Box`, follows `b = Box(...)` to `b.center`, and knows a function's keyword arguments before that function exists anywhere.

**The kernel** answers against the live namespace — the same mechanism that gives IPython its Tab completion. It knows the methods of the part you built two cells ago, which no reading of the file can, because that object does not exist in the file: it exists in a kernel that has run it.

Neither alone is enough. On a cold kernel a file beginning `from build123d import *` has nothing live to offer; on a half-written file the static half cannot see what a cell already produced. Where both have a name, the kernel's entry wins, because it carries the signature IPython worked out.

The same connection answers the other two questions the editor asks: **hover** for what a name actually holds — which build123d raises constantly, since operations return `Part`, `Sketch`, `Curve` and `Compound` almost interchangeably — and **parameter hints** for the call the caret is inside, with the argument you are typing highlighted. `Box(10, 10, 10)` is three numbers that mean length, width and height, and nothing else on screen says so.

## Squiggles

The language server publishes what it thinks is wrong with the buffer, and the editor underlines it. This is pushed rather than asked for, so it refreshes as you type rather than when you happen to open the suggestion list.

The value is being told before you run it. _"AttributeError: 'Cube' object has no attribute 'part'"_ is a true and useless sentence once the file has already run for thirty seconds and tessellated something; the same fact underlined as you type costs nothing.

## Formatting

The formatter is [ruff](https://docs.astral.sh/ruff/), run in the sidecar.

- `Shift-Alt-F`, or **Format Document** from the palette or the context menu, formats the buffer.
- **Format on save** is on by default. ruff changes layout and never meaning, the result is one undo away, and a buffer that does not parse is left exactly as it is rather than half done.
- **Line length** in **Settings → Editor** is both the width ruff wraps at and the column the editor's ruler is drawn at, so the line you are looking at is the line the formatter will break. The default is 88, which is ruff's own.

ruff is run isolated, so the number in Settings is the number used — a project's own `pyproject.toml` or `ruff.toml` does not quietly change the width the editor promised.

Above **50 000 lines** a buffer is saved without being formatted, and the log says so. ruff itself is not the reason — it formats a million lines in well under a second — it is that a file that size gets no [recovery copy](#saving), so its edits exist nowhere but in memory and the save is written immediately rather than held open. No Python anybody writes comes near it; generated files do.

## Find

`Cmd`/`Ctrl-F` opens Monaco's find widget. `Alt-Enter` there selects **every** match at once, as it does in VS Code, so the next thing typed replaces all of them.

That chord is **Run All** everywhere else in the editor. While the find widget is open it belongs to the find widget, which is what everybody who has used another editor expects it to do.

## Snippets

Type a prefix and accept the suggestion. The shipped set is the build123d CodeCAD speedmodeling snippets by Jern, from [build123d-portable](https://github.com/build123d/build123d-portable), with the viewer import pointed at `build123d_studio`.

Their prefixes begin with a punctuation mark on purpose, so they cannot collide with a real identifier the language server would also suggest — and the mark says which style the snippet is for: `?` is builder mode, `>` is algebra mode.

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

Twenty-five in all; typing `?` or `>` on its own narrows the list to one family.

Your own snippets go in `snippets.json` beside the settings, in the [data directory](first_run.md#where-it-all-lives). It is VS Code's `.code-snippets` shape, read with the same parser VS Code uses — so comments and trailing commas are fine, and a `.code-snippets` file copied in unedited works. An entry with the same prefix as a shipped one replaces it, so the shipped set can be overridden without being edited.

## The new-file template

**Settings → New file** holds what a new buffer starts with. Cleared, **New File** opens an empty buffer; **Restore default** puts the shipped template back.

It is a snippet, in the same syntax: `$1` and `${1:like this}` are stops to tab between, `${1|Box,Cylinder,Sphere|}` offers a choice, `$0` is where the caret ends, and `$CURRENT_YEAR`, `$RANDOM` and `$CLIPBOARD` are filled in. A literal dollar is `\$`.

## The command palette

`F1` opens Monaco's own palette, which lists every editor command by name — find, replace, comment toggling, folding, formatting, and the run and debug commands this application added. It comes from Monaco rather than from the [keymap](commands.md), so it is not editable in Settings, and it only works while the caret is in the editor. The toolbar button works from anywhere and focuses the editor first.

## Saving

A save writes **into** the file rather than replacing it, so it stays the same file: a hard link to it still sees the new contents, and extended attributes — Finder tags among them on macOS — ownership and permissions all survive. Symlinks are followed to the file they actually point at, so a `part.py` symlinked into a repository stays a symlink and the real file keeps receiving the edits.

What that gives up is atomicity, and what replaces it is the recovery copy below: a save interrupted half-way leaves the work in the journal rather than in a temporary file. A buffer too large to be journalled continuously is copied there once, immediately before the write.

If the file changed underneath you — a formatter, a `git checkout`, a second window — you are asked before anything is overwritten. That question now arrives when you **focus the window or choose the tab**, not only when you try to save, so a file rewritten elsewhere stops being read as current. **Reload** takes what is on disk, **Overwrite** writes your buffer over it, and **Cancel** marks the tab modified so the choice survives being closed.

Unsaved buffers are shadowed into the data directory as you type, and the copy is removed the moment it stops being the only one — a successful save, a closed tab, a graceful quit. Every ordinary way out already prompts; the shadow copy is for the ends that ask nobody, a webview that dies or a power cut. What is left over at the next start therefore means exactly one thing, and you are offered it back. The copies are plain source files, so they can be opened in any editor while the application is not running.

## Opening files

Every file in the tree can be opened, not only Python. Only `.py` and `.pyi` are treated as Python: anything else is plain text, so a STEP export opened to be looked at is neither highlighted as Python, sent to the language server, nor offered to the formatter. Two things are refused or questioned rather than attempted: anything that is not text — one NUL byte in the first 8 kB, which is git's own test — and anything over 10 MB, which asks first. The limit is about the language server rather than the editor: Monaco tokenises the viewport rather than the document and handles a 32 MB file comfortably, but the whole buffer is sent for analysis on open and after every edit.
