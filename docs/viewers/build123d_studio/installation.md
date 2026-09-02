# Installation

build123d Studio is a desktop application for macOS, Windows and Linux: a Monaco editor, the [CAD Viewer](../../viewer.md), a real Jupyter console and a variable explorer in one window — and it brings its own Python. Nothing to `pip install`, no virtualenv to activate, no interpreter to choose.

## Download and verify

Download the latest release from [the Releases page](https://github.com/bernhard-42/build123d-studio/releases), compute the checksum of the downloaded file

- macOS: `shasum -a 256 build123d-studio-*.dmg`
- Windows: `certutil -hashfile build123d-studio-*.zip SHA256`
- Linux: `sha256sum build123d-studio-*.AppImage`

and compare it against the values in `SHA256SUMS.txt` on the same Releases page. With the GitHub CLI [gh](https://cli.github.com/) installed, the build provenance can be verified directly:

```sh
gh attestation verify build123d-studio-*.dmg --repo bernhard-42/build123d-studio
```

## Install

The ordinary way for your platform: drag the `.app` to Applications on macOS, unzip the Windows package, store the Linux AppImage where you like it. The application's own directory is never written to, so it can be installed read-only.

Then continue with [First Run](first_run.md) — the first start builds the Python environment and takes a few minutes.

## The `studio` command

A small launcher ships with every release. Copy it somewhere on your `PATH`, and `studio`, `studio .`, `studio some/folder` or `studio part.py` opens that path in a new window — a file opens its containing folder as the project.

Any directory on your `PATH` will do — `~/.local/bin` is the usual choice on macOS and Linux. Check yours first, since it is only on the `PATH` by default on some systems:

```bash
mkdir -p ~/.local/bin
case ":$PATH:" in *":$HOME/.local/bin:"*) echo "on PATH" ;; *) echo "NOT on PATH" ;; esac
```

Where the launcher comes from differs by platform, because the packages do:

=== "macOS"

    Inside the application bundle:

    ```bash
    install -m 755 "/Applications/build123d Studio.app/Contents/MacOS/studio" ~/.local/bin/
    ```

=== "Windows"

    In the folder you unzipped. `~/.local/bin` means nothing to `cmd.exe`, so copy it to a directory that is already on your `PATH` — `$env:PATH -split ";"` in PowerShell lists them:

    ```bat
    copy C:\path\to\build123d-studio\studio.cmd C:\some\folder\on\PATH\
    ```

=== "Linux"

    A **separate download** called `studio`, beside the AppImage on the Releases page — an AppImage is a single read-only file, so there is nothing to copy it out of:

    ```bash
    install -m 755 studio ~/.local/bin/
    ```

    Releases up to and including 0.5.1 do not have that download. Extract the launcher from the AppImage instead — it is the same file:

    ```bash
    ./build123d-studio-*.AppImage --appimage-extract usr/bin/studio
    install -m 755 squashfs-root/usr/bin/studio ~/.local/bin/
    ```

**Run the application once first.** The launcher does not work out where the application is from its own location — once copied it has no relationship to the package it came from. The application records its own location on every start instead, and the script reads that. Which also means moving the application fixes itself: start it once from wherever it is now.

Every form starts a new instance, so several projects can be open at once. That matters most on macOS, where double-clicking an already-running application activates the existing window rather than starting a second one.
