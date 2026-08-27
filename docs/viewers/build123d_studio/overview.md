# Overview

build123d Studio is a desktop application for macOS, Windows and Linux: a code editor, the [CAD Viewer](../../viewer.md), a real Jupyter console and a variable explorer in one window. It brings its own Python — nothing to `pip install`, no virtualenv to activate, no interpreter to choose; the first start builds a private environment with build123d and everything else it needs.

![](../../assets/build123d-studio.png#only-light)
![](../../assets/build123d-studio-dark.png#only-dark)

## Key features

- The complete [CAD Viewer](../../viewer.md) beside the editor: navigation tree, clipping and zebra tools, measurement and analysis tools, [materials](../../pbr_studio.md) and [animation](../../animation.md).
- Batteries included: one release pins one [uv](https://docs.astral.sh/uv/) and one CPython, so the environment it builds is the same on every machine and never touches any other Python — see [First Run](first_run.md).
- A full editor with code completion, type checking and formatting, backed by a language server running in the same environment as your code.
- A Jupyter console and a variable explorer on the kernel your code runs on, for inspecting the objects a script created.
- Measurements computed in their own process beside the kernel, so analysing a model never competes with running code — see [Concepts](concepts.md).
- The `studio` launcher: `studio some/folder` or `studio part.py` opens a project from the command line — see [Installation](installation.md#the-studio-command).

Continue with [Installation](installation.md).
