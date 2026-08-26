# First Run

## Getting past the OS gatekeepers

- **macOS**: builds are signed ad hoc, with no Developer ID and no notarisation, so the first run is blocked with *"Apple could not verify 'build123d Studio' is free of malware"*. Open **System Settings → Privacy & Security**, find *"build123d Studio" was blocked to protect your Mac* under _Security_, and click **Open Anyway**. Alternatively, run `xattr -dr com.apple.quarantine "/Applications/build123d Studio.app"` before double-clicking the icon.
- **Windows**: builds are unsigned, so SmartScreen warns on first run. Use **More info → Run anyway**.
- **Linux**: the app uses the system webview and needs WebKitGTK (`libwebkit2gtk-4.1-0`, or `libwebkit2gtk-4.0-37` on older releases) — install it if it is not already there. Mark the AppImage executable (`chmod +x build123d-studio-*.AppImage`), or the desktop treats it as a data file.

## The first start

The first start takes a few minutes and shows what it is doing: a splash overlay reports each step while it downloads a pinned [uv](https://docs.astral.sh/uv/), downloads a pinned CPython, builds a virtual environment of its own and installs build123d and everything else it needs — several hundred megabytes in total, including the language server and the OpenCascade bindings. It never touches a system or user Python, and nothing it does is visible to any other Python on the machine. One release pins one uv and one CPython, so two people running the same release get the same interpreter and the same packages.

Everything it writes lives in one per-user **data directory**, holding the environment, the settings and the log:

- macOS: `~/Library/Application Support/build123d-studio/`
- Windows: `%APPDATA%\build123d-studio\`
- Linux: `~/.local/share/build123d-studio`

Every start after the first goes straight to the window: the environment is reconciled against the locked package list while the kernel warms in the background — the kernel indicator in the toolbar says `starting`, then `idle`, and you can type before it gets there.

Uninstalling the app does not remove the environment; the path is in the About dialog and in the log.

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
