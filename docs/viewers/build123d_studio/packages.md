# Packages and the environment

Studio builds its own Python environment on [first run](first_run.md) and manages it from **Settings → Packages**. It is the equivalent of the VS Code viewer's [Library Manager](../ocp_vscode/managers.md), with one difference: there is no interpreter to choose. There is one, and the application owns it.

## What is fixed and what is not

Two packages have a source picker:

- **build123d** — PyPI (the released version), the `dev` branch on GitHub, or a local checkout
- **ocp-viewer-core** — PyPI, the `main` branch on GitHub, or a local checkout

Everything else the application declares is frozen to the versions this release was built and tested against. `ocp-viewer-core` matches the three-cad-viewer build in the frontend, and its metadata sets `ocp-tessellate`'s range.

!!! warning

    The GitHub option builds and runs whatever is on that branch at the moment it is fetched — unreleased code. It needs `git`; without it the option is disabled and says so. The check runs `git --version` when Settings opens and believes git's own banner over the exit code; a check that could not tell — a shell that did not answer — is asked again next time rather than remembered, and a stored GitHub choice is only reverted to PyPI when git is confirmed absent.

A **local checkout** is installed editable: what you edit is what runs. Choose the folder with **Choose…**, which also selects the Local radio. Editable installs use the layout static analysis can follow, so a local checkout resolves in [completion and squiggles](editor.md) as well as at runtime.

## The tab

Top to bottom:

**What this release declares** — the `app` and `core_cad` groups as the environment has them, alphabetically: which version of what you are running.

**Customization** — the source pickers and the [additional packages](#additional-packages) field, with **Update packages** under them.

**Upgrade** and **Restore** — see below.

| Button               | What it does                                                                                                                                                                     |
| -------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Apply**            | Saves every field and closes. No uv, no kernel restart.                                                                                                                          |
| **Update packages**  | Saves, then makes the environment match what is declared: `uv sync`. Adds what is new, picks up a changed source, leaves every already-locked package as it is.                  |
| **Upgrade packages** | Saves, then re-locks every package declared with a range — each as far as the range allows — and syncs. See [what an upgrade moves](#what-an-upgrade-moves-and-what-it-leaves). |
| **Restore packages** | Puts the environment back to what this release ships. See [Restore](#restore).                                                                                                   |

**Only Apply and Cancel close the dialog.** Update, Upgrade and Restore act on the environment and return to the dialog, on success and on failure.

Each of the three runs behind the splash with uv's own output on screen. When uv has changed anything, the libraries are loaded once while the splash is up — `import OCP`, `import build123d`, and `import cadquery` if you have it — each announced before it begins; the three take about two minutes on an Apple M1 the first time after a change, because macOS verifies OpenCascade's signed libraries whenever they are replaced. An update that changed nothing skips this. Then the language server and the kernel restart. The splash waits for you to dismiss it; after a failure it is where the reason is.

Saving without installing is fine: the application runs `uv sync` on every launch, so a change that was applied but not installed lands at the next start.

### The environment's `pyproject.toml`

Studio keeps a uv project in the [data directory](first_run.md#where-it-all-lives): an ordinary `pyproject.toml`. Three regions are rewritten without asking, and the file says so at the top:

| Region                          | Written by                                           |
| ------------------------------- | ---------------------------------------------------- |
| the `app` and `core_cad` groups | replaced when you install a new build123d Studio     |
| the `user` group                | Settings → Additional packages, when you press Apply |
| `[tool.uv.sources]`             | Settings → the source pickers, when you press Apply  |

Settings fills its fields from that file when it opens. If the file has been edited in a way the dialog cannot read, it says so.

!!! warning "Adding a package with uv"

    **Don't edit pyproject.toml by hand. There is no guarantee the app will work afterwards.**

    If you do: `uv add <package>` writes into `[project].dependencies`, not into a group, so `Upgrade packages` — which moves the groups — does not touch it. Use `uv add --group user <package>` to put it where Settings and Upgrade expect it. Either way it survives a new release.

### What an upgrade moves, and what it leaves

`Upgrade packages` moves the three groups, each package as far as its own range allows:

| Declared as                                      | On Upgrade                               |
| ------------------------------------------------ | ---------------------------------------- |
| `build123d>=0.11.1`                              | any newer release                        |
| `ocp-viewer-core>=1.0.5,<1.1.0`                  | within its minor                         |
| `cadquery-ocp-stubs>=7.9.3,<7.10`                | within the OpenCascade line it describes |
| `ipykernel>=7.3.0,<7.4.0`, and the rest of `app` | within their patch level                 |
| `basedpyright==1.39.9`, `ruff==0.16.3`           | not at all                               |
| whatever you put in `user`                       | as far as what you wrote allows          |

Three packages ocp-viewer-core brings with it — `ocp-tessellate`, `pillow` and `threejs-materials` — move as well: a fix in the tessellator is a fix in what you see.

Everything else is build123d's own dependency tree — `numpy`, `scipy`, `fonttools` and some forty more — and moves when build123d moves it.

Nothing overwrites the environment's lockfile once it exists.

### Restore

At the foot of the tab. It puts `pyproject.toml` and `uv.lock` back to what the release ships and installs exactly that: build123d and ocp-viewer-core from PyPI, additional packages cleared, edits to those two files gone. The result is the environment a first start builds.

## Additional packages

One requirement per line. The line says where the package comes from:

```text
mylib>=0.3                                    PyPI, with or without a version range
/Users/me/src/mylib                           a local checkout, installed editable
git+https://github.com/someone/mylib@main     a git repository
mylib @ git+https://github.com/x/python-lib   named explicitly
```

Give a local checkout its full path; a relative path such as `../src/mylib` is refused.

The package name is taken from the last path segment when it is not given. When the distribution is not named after its folder — `python-foo` shipping `foo` — use the `name @ source` form.

Two rules:

- **A package the application already declares cannot be named**: `build123d`, `ocp-viewer-core`, `jupyter_console`, `orjson`, `websockets`, `pywinpty` and `basedpyright`. uv intersects duplicate declarations, so such an entry would do nothing, or make the environment unsolvable.
- **The same package cannot be given twice.**

Packages those bring in — `ocp-tessellate`, `ipykernel`, `prompt_toolkit` and the rest — can be constrained here; uv enforces the parent's range on top of what you write.

Whether a version exists, whether a combination resolves, whether a URL is reachable is uv's to answer. A line that breaks the environment does not stop the application from starting: that section is dropped for the launch, and the text stays in Settings to be corrected.

## Where it all lives

The environment and the lockfile are in the [environment root](first_run.md#where-it-all-lives); the settings and the logs are beside them on macOS and Linux, and in the roaming half of `AppData` on Windows, where the environment is in the local half. `BUILD123D_STUDIO_ENV_ROOT` [moves the environment](first_run.md#moving-the-environment) anywhere, for a machine whose policy will not execute binaries out of a user-writable path. The application's own directory is never written to, so it can be installed read-only; uninstalling does not remove the environment. **Help → About** shows the environment path — and says when that variable chose it — with the uv version, the interpreter and every log file: what a bug report should carry.

Nothing here is visible to any other Python on the machine, and no system or user Python is touched.
