# First Run

## Getting past the OS gatekeepers

- **macOS**: builds are signed ad hoc, with no Developer ID and no notarisation, so the first run is blocked with _"Apple could not verify 'build123d Studio' is free of malware"_. Open **System Settings → Privacy & Security**, find _"build123d Studio" was blocked to protect your Mac_ under _Security_, and click **Open Anyway**. Alternatively, run `xattr -dr com.apple.quarantine "/Applications/build123d Studio.app"` before double-clicking the icon.
- **Windows**: builds are unsigned, so SmartScreen warns on first run. Use **More info → Run anyway**.
- **Linux**: Mark the AppImage executable (`chmod +x build123d-studio-*.AppImage`), or the desktop treats it as a data file. Furthermore, the app uses the system webview and needs WebKitGTK (`libwebkit2gtk-4.1-0`, or `libwebkit2gtk-4.0-37` on older releases) — install it if it is not already there.

## The first start

The first start takes a few minutes and shows what it is doing: a splash overlay reports each step while it downloads a pinned [uv](https://docs.astral.sh/uv/), downloads a pinned CPython, builds a virtual environment of its own and installs build123d and everything else it needs — several hundred megabytes in total, including the language server and the OpenCascade bindings. It never touches a system or user Python, and nothing it does is visible to any other Python on the machine. One release pins one uv and one CPython, so two people running the same release get the same interpreter and the same packages.

Every start after the first goes straight to the window: the environment is reconciled against the locked package list while the kernel warms in the background — the kernel indicator in the toolbar says `starting`, then `idle`, and you can type before it gets there.

Uninstalling the app does not remove the environment; the path is in the About dialog and in the log.

## Where it all lives

Everything the application writes is per-user, and none of it goes into the application's own folder — so an installation can be read-only, and replacing it does not disturb anything.

| What                                              | macOS                                                    | Windows                                   | Linux                                     |
| ------------------------------------------------- | -------------------------------------------------------- | ----------------------------------------- | ----------------------------------------- |
| **The environment** — interpreter, venv, uv cache | `~/Library/Application Support/build123d-studio/runtime` | `%LOCALAPPDATA%\build123d-studio\runtime` | `~/.local/share/build123d-studio/runtime` |
| **Settings, snippets, logs, recovery copies**     | `~/Library/Application Support/build123d-studio/`        | `%APPDATA%\build123d-studio\`             | `~/.local/share/build123d-studio/`        |

On Windows the two are deliberately different halves of `AppData`. The environment is around 1.8 GB across some 38,000 files, and `%APPDATA%` — Roaming — is copied at logon or redirected to a network share on a managed machine, which is exactly what such a tree must not be. `%LOCALAPPDATA%` is excluded from roaming by design. Settings and snippets stay in Roaming, where a preference that should follow you between machines belongs.

!!! note "Upgrading from an earlier version on Windows"

    Earlier releases built the environment in `%APPDATA%\build123d-studio\runtime`. The new location starts empty, so the first start after the upgrade rebuilds it — a download, once. Nothing is moved, because a virtualenv records absolute paths and a moved one is a broken one. The old folder is left where it is and named in the log; it is safe to delete.

## Moving the environment

Some machines will not execute a binary out of a user-writable directory at all. That is a deliberate policy — AppLocker and WDAC deny `%APPDATA%`, `%LOCALAPPDATA%` and `%TEMP%` in their recommended rule sets, and a `noexec` mount over `/home` is the same idea on Linux — and Studio runs uv, a CPython and the OpenCascade extension modules from inside its environment, so it is exactly what such a rule stops.

Set **`BUILD123D_STUDIO_ENV_ROOT`** to an absolute path and the environment is built there instead, on every platform. Nothing else moves: settings, snippets and logs stay where they are.

- The path must be absolute. `../env` and `~/env` are refused — the value is read from the environment of whatever launched the application, which is a desktop session rather than a shell, so there is nothing dependable for them to be relative to.
- Missing directories are created, parents included.
- If it is set and the directory cannot be created, the application says so and stops rather than quietly rebuilding in the place you were avoiding.
- **Help → About** shows the location, and says when `BUILD123D_STUDIO_ENV_ROOT` is what chose it — so a bug report carries the answer.

Getting the variable to the application differs by platform, and only one of the three is awkward:

=== "Windows"

    `start` passes the console's environment through, so the launcher works:

    ```bat
    set BUILD123D_STUDIO_ENV_ROOT=D:\studio-env
    studio
    ```

    To make it permanent, set it as a user environment variable (**System Properties → Environment Variables**), or machine-wide by policy.

=== "Linux"

    The launcher `exec`s the application, so the variable is inherited:

    ```bash
    BUILD123D_STUDIO_ENV_ROOT=/opt/studio-env studio
    ```

=== "macOS"

    `studio` launches through `open`, and LaunchServices does not pass the calling shell's environment on. Run the executable inside the bundle directly:

    ```bash
    BUILD123D_STUDIO_ENV_ROOT=/opt/studio-env \
      "/Applications/build123d Studio.app/Contents/MacOS/build123d-studio"
    ```

    For a value that also applies when the application is started from the Finder, `launchctl setenv BUILD123D_STUDIO_ENV_ROOT /opt/studio-env` puts it in the login session — applications started after that inherit it, and it is forgotten at logout unless something sets it again.

!!! warning "It has to be a directory the policy actually allows"

    Pointing this at another user-writable folder changes nothing: the rules are about which paths may be executed from, not about which application wrote them. Somewhere an administrator has allowed is the point of the setting. Putting it inside the application's own folder is possible but has its own cost — the environment is then destroyed and rebuilt by every update, which is precisely what keeping it outside avoids.

## First model

```python
from build123d import *
from build123d_studio import show

b = Box(1, 2, 3)
show(b)
```

Run it with `Shift-Enter` (cell by cell on the kernel) or `Ctrl-F5` (the whole file in its own process) — new files start from a template with the imports already in place.

## Troubleshooting

If something goes wrong, consult the log in the data directory above. How much it captures is **Settings → Application → Debug console**; the exact file paths are in **Help → About**.
