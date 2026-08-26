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

A small launcher ships beside the application. Copy it somewhere on your `PATH`, and `studio`, `studio .`, `studio some/folder` or `studio part.py` opens that path in a new window — a file opens its containing folder as the project. Run the application once first: it records its own location on every start, which is how the script finds it.
