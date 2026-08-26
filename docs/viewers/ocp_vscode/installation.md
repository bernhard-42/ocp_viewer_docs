# Installation

The VS Code CAD Viewer is the extension **OCP CAD Viewer** plus the Python package **ocp_vscode**, installed at matching versions.

## The extension

Install "OCP CAD Viewer" from the [VS Code Marketplace](https://marketplace.visualstudio.com/items?itemName=bernhard-42.ocp-cad-viewer) (or via the Extensions view, `Ctrl-Shift-X` / `Cmd-Shift-X`). It adds the _OCP CAD Viewer_ icon to the activity bar, with the **Viewer Manager** and **Library Manager** panels.

## Quickstart

The fastest way to a working setup are the **Quickstart build123d** and **Quickstart CadQuery** buttons in the sidebar. One click:

1. installs the chosen CAD library (`build123d` or `cadquery`) plus `ocp_vscode`, `ocp_tessellate` and `ipykernel` into the currently selected Python environment, using the commands from `OcpCadViewer.advanced.quickstartCommands`
2. optionally installs the [Jupyter extension for VS Code](https://marketplace.visualstudio.com/items?itemName=ms-toolsai.jupyter)
3. starts the viewer
4. creates a demo file in a temporary folder, so there is immediately a working example on screen

## The Library Manager

The **Library Manager** in the sidebar manages the Python libraries for _build123d_, _cadquery_, _ipykernel_ and _ocp_tessellate_. Hover over a library and press the down-arrow to install or upgrade it; VS Code prompts to confirm the selected environment. The paste button next to each installed library inserts its import snippet at the cursor.

The commands it runs come from two settings and can be customized:

```json
  "OcpCadViewer.advanced.quickstartCommands": {
    "cadquery":  ["{unset_conda} {pip-install} ocp_vscode~={ocp_vscode_version} cadquery"],
    "build123d": ["{pip-install} ocp_vscode~={ocp_vscode_version} git+https://github.com/gumyr/build123d"]
  },
  "OcpCadViewer.advanced.installCommands": {
    "cadquery":        ["{unset_conda} {pip-install} --upgrade cadquery"],
    "build123d":       ["{pip-install} --upgrade git+https://github.com/gumyr/build123d"],
    "cadquery_ocp":    ["{pip-install} --upgrade cadquery_ocp"],
    "ocp_vscode":      ["{pip-install} --upgrade ocp_vscode~={ocp_vscode_version}"],
    "ocp_tessellate":  ["{pip-install} --upgrade ocp_tessellate"],
    "ipykernel":       ["{pip-install} --upgrade ipykernel"],
    "jupyter_console": ["{pip-install} --upgrade jupyter_console"]
  },
```

Notes:

- Due to rapid development of build123d, the default install commands use the installation from git. Change to PyPI if you want the latest published release instead.
- `uv` environments are auto-detected and `{pip-install}` becomes `uv pip install`. To use `uv add` instead, override the commands, e.g. `"build123d": ["uv add -p {python} --upgrade git+https://github.com/gumyr/build123d"]`.

### Placeholders

Commands may contain the following placeholders, resolved at runtime against the currently selected Python interpreter:

| Placeholder | Replaced with |
| ----------- | ------------- |
| `{python}` | Absolute path to the selected interpreter (quoted) |
| `{pip-install}` | `uv pip install -p "{python}"` if the env is a `uv` venv, otherwise `"{python}" -m pip install` |
| `{unset_conda}` | On macOS/Linux, prefix `env -u CONDA_PREFIX`; on Windows, a temporary `.cmd` file that unsets `CONDA_PREFIX` before running the command |
| `{ocp_vscode_version}` | Version of the currently installed VS Code extension |

## Downloading examples

In the **Library Manager**, find "examples (download)" under an installed library (not every library has examples) and press the green download button. Check and adjust the target path if needed — a file needs to be open so the extension knows the working folder. The examples land in a folder named `<library>_examples`, e.g. `cadquery_examples`.

## Version matching

Extension and Python package must have the same version; the viewer UI and the Output panel report a mismatch:

```text
extension.check_upgrade: ocp_vscode library version 2.8.6 matches extension version 2.8.6
```

The command "Install `ocp_vscode` library" (also offered automatically) installs the Python side at the version the extension expects.

## Troubleshooting installs

- If the install message reports a missing `pip` or `uv pip`, confirm the selected Python interpreter actually has one of them available.
- For conda/mamba/micromamba environments, `pip` (or `uv pip`) must be used — `ocp_vscode` is only published on PyPI, not on conda channels. The default commands handle this via `{unset_conda}`.
- If `python` cannot be found at all, restart VS Code after activating your virtual environment in the terminal, or pick the interpreter via the Python extension's "Select Interpreter" command.
