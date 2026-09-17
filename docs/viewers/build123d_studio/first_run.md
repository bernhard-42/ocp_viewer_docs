# First Run

## Getting past the OS gatekeepers

- **macOS**: builds are signed ad hoc, with no Developer ID and no notarisation, so the first run is blocked with _"Apple could not verify 'build123d Studio' is free of malware"_. Open **System Settings → Privacy & Security**, find _"build123d Studio" was blocked to protect your Mac_ under _Security_, and click **Open Anyway**. Or run `xattr -dr com.apple.quarantine "/Applications/build123d Studio.app"` before double-clicking the icon.
- **Windows**: builds are unsigned, so SmartScreen warns on first run. Use **More info → Run anyway**.
- **Linux**: mark the AppImage executable (`chmod +x build123d-studio-*.AppImage`). The app uses the system webview and needs WebKitGTK (`libwebkit2gtk-4.1-0`, or `libwebkit2gtk-4.0-37` on older releases).

## The first start

The first start takes a few minutes. A splash overlay, headed with the version, reports each step: a pinned [uv](https://docs.astral.sh/uv/) is downloaded, then a pinned CPython, a virtual environment is built and build123d and everything else is installed — several hundred megabytes, including the language server and the OpenCascade bindings. No system or user Python is touched, and nothing is visible to any other Python on the machine. One release pins one uv and one CPython, so two people running the same release get the same interpreter and packages.

Every later start goes straight to the window. The environment is reconciled against the locked package list while the kernel warms up in the background; the kernel indicator says `starting`, then `idle`, and you can type before it gets there.

Uninstalling the app does not remove the environment; its path is in About and in the log.

## Where it all lives

Everything the application writes is per-user, and none of it goes into the application's own folder, so an installation can be read-only and replacing it disturbs nothing.

| What                                              | macOS                                                    | Windows                                   | Linux                                     |
| ------------------------------------------------- | -------------------------------------------------------- | ----------------------------------------- | ----------------------------------------- |
| **The environment** — interpreter, venv, uv cache | `~/Library/Application Support/build123d-studio/runtime` | `%LOCALAPPDATA%\build123d-studio\runtime` | `~/.local/share/build123d-studio/runtime` |
| **Settings, snippets, logs, recovery copies**     | `~/Library/Application Support/build123d-studio/`        | `%APPDATA%\build123d-studio\`             | `~/.local/share/build123d-studio/`        |

On Windows the environment — about 1.8 GB in some 38,000 files — is in the local half of `AppData`, which is not copied at logon or redirected to a network share on a managed machine. Settings and snippets are in the roaming half, where a preference that follows you between machines belongs.

!!! note "Upgrading from an earlier version on Windows"

    Earlier releases built the environment in `%APPDATA%\build123d-studio\runtime`. The first start after the upgrade builds it afresh in the new location — one download. The old folder is left in place and named in the log; it is safe to delete.

## Moving the environment

Some machines do not execute binaries from a user-writable directory: AppLocker and WDAC deny `%APPDATA%`, `%LOCALAPPDATA%` and `%TEMP%` in their recommended rule sets, and a `noexec` mount over `/home` does the same on Linux. Studio runs uv, a CPython and the OpenCascade extension modules from inside its environment, so such a rule stops it.

Set **`BUILD123D_STUDIO_ENV_ROOT`** to an absolute path and the environment is built there instead, on every platform. Settings, snippets and logs stay where they are.

- The path must be absolute; `../env` and `~/env` are refused.
- Missing directories are created, parents included.
- If the directory cannot be created, the application says so and stops.
- **Help → About** shows the location and says when `BUILD123D_STUDIO_ENV_ROOT` chose it.

Getting the variable to the application differs by platform:

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

    `studio` launches through `open`, and LaunchServices does not pass the shell's environment on. Run the executable inside the bundle directly:

    ```bash
    BUILD123D_STUDIO_ENV_ROOT=/opt/studio-env \
      "/Applications/build123d Studio.app/Contents/MacOS/build123d-studio"
    ```

    For a value that also applies when the application is started from the Finder, `launchctl setenv BUILD123D_STUDIO_ENV_ROOT /opt/studio-env` puts it in the login session; applications started afterwards inherit it, and it is forgotten at logout.

!!! warning "It has to be a directory the policy allows"

    Pointing this at another user-writable folder changes nothing: the rules are about which paths may be executed from. Use a directory an administrator has allowed. Inside the application's own folder works but the environment is then rebuilt by every update.

## First model

```python
from build123d import *
from build123d_studio import show

b = Box(1, 2, 3)
show(b)
```

Run it with `Shift-Enter` (cell by cell on the kernel) or `Ctrl-F5` (the whole file in its own process). New files start from a template with the imports in place.

## Troubleshooting

If something goes wrong, consult the log in the data directory above. How much it captures is **Settings → Application → Debug console**; the exact file paths are in **Help → About**.
