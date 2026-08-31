# Packages and the environment

Studio builds its own Python environment on [first run](first_run.md) and manages it from **Settings → Packages**. This page is what that tab does. It is the equivalent of the VS Code viewer's [Library Manager](../ocp_vscode/managers.md), with one difference that shapes everything else: there is no interpreter to choose, because there is only one and the application owns it.

## What is fixed and what is not

Two packages have a source picker:

- **build123d** — PyPI (the released version), the `dev` branch on GitHub, or a local checkout
- **ocp-viewer-core** — PyPI, the `main` branch on GitHub, or a local checkout

Everything else the application declares is frozen to the versions this release was built and tested against. `ocp-viewer-core` is tied to the three-cad-viewer build in the frontend — both halves ship as one version — and it owns `ocp-tessellate`'s range through its own metadata, so a picker for the rest would only offer ways to break the viewer.

!!! warning

    The GitHub option builds and runs whatever is on that branch at the moment it is fetched, which is unreleased code from the internet. It is opt-in, and it needs `git`: without it the option is disabled and says so.

A **local checkout** is installed editable, so what you edit is what runs. Choose the folder with the **Choose…** button — picking one also selects the Local radio, because choosing a folder and then finding it had no effect is the obvious way for this to be annoying. Editable installs are made in the layout static analysis can follow, so a local checkout resolves in [completion and squiggles](editor.md) as well as at runtime.

## The buttons

**Only Apply and Cancel close the dialog.** Install, Re-install and Upgrade act on the environment and return to it, on success as well as on failure — so there is never a question of what happened or where you now are.

| Button                     | What it does                                                                                                                                                                                                                                                                      |
| -------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Apply**                  | Saves every field and closes. Touches nothing else — no uv, no kernel restart.                                                                                                                                                                                                    |
| **Install packages**       | Saves, then makes the environment match what is declared: `uv lock`, then `uv sync`. Adds what is new and leaves every already-locked package where it is.                                                                                                                        |
| **Re-install `<package>`** | Saves, then `uv sync --reinstall-package <package>`. There is one per picker. A local checkout is editable, so ordinary code edits are already live and this is not needed for them — it is for when the package's own metadata changes, such as a new dependency or entry point. |
| **Upgrade packages**       | Saves, then `uv lock --upgrade`, then `uv sync`. Every package moves as far as its own version range allows: build123d to any newer release, ocp-viewer-core within its minor. Everything else is pinned exactly and cannot move at all.                                          |

Each of the three runs behind the splash with uv's own output on screen, and each is followed by two restarts: the language server, whose index otherwise describes an environment that no longer exists, and the kernel, which has already imported whatever was replaced. The splash waits for you to dismiss it, because after a failure it is the only place that says why.

Saving without installing is coherent: the application runs `uv sync` on every launch, so a change that was applied but not installed simply lands at the next start.

## Additional packages

One requirement per line. The line itself says where the package comes from — there is no source picker, because the string _is_ the source:

```text
ocp-widgets>=0.3                              PyPI, with or without a version range
/Users/me/src/mylib                           a local checkout, installed editable
git+https://github.com/someone/mylib@main     a git repository
mylib @ git+https://github.com/x/python-lib   named explicitly
```

Give a local checkout its full path. This is a dialog rather than a shell, so `../src/mylib` has nothing to be relative to and is refused — uv would resolve it against the environment directory, and the bad case is not the one that fails but the one that finds a different directory of the same name.

The package name is taken from the last path segment when it is not given, which is right for most repositories and wrong whenever the distribution is not named after its folder — `python-foo` shipping `foo`. The `name @ source` form is how to say so.

Two rules are enforced here, and only two:

- **A package the application already declares cannot be named.** Those are `build123d`, `ocp-viewer-core`, `jupyter_console`, `orjson`, `websockets`, `pywinpty` and `basedpyright`. uv _intersects_ duplicate declarations rather than letting the later one win, so such an entry would silently do nothing — or, asking for an older version, produce an environment that cannot be solved instead of the version you asked for.
- **The same package cannot be given twice.**

Anything those two lists bring in is fair game: `ocp-tessellate`, `ipykernel`, `prompt_toolkit` and the rest arrive under a parent that owns their range, and constraining one of them is yours to do — uv enforces the parent's range on top of whatever you write.

Everything else — whether a version exists, whether a combination resolves, whether a URL is reachable — is uv's to answer, and it answers better than a rule here would. A line that breaks the environment does not prevent the application from starting: the failed section is dropped for that launch, with the text left in Settings to be corrected.

## Where it all lives

The environment and the lockfile live in the [environment root](first_run.md#where-it-all-lives), the settings and the logs beside them — the same directory on macOS and Linux, and the local rather than the roaming half of `AppData` on Windows. `BUILD123D_STUDIO_ENV_ROOT` [moves the environment](first_run.md#moving-the-environment) anywhere, for a machine whose policy will not execute binaries out of a user-writable path. The application's own directory is never written to, so it can be installed read-only, and uninstalling does not remove the environment. **Help → About** names the environment path — and says when that variable is what chose it — along with the uv version, the interpreter and every log file. It is what a bug report should carry.

Nothing here is visible to any other Python on the machine, and no system or user Python is touched.
