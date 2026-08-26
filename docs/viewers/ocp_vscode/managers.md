# Viewer Manager and Library Manager

The _OCP CAD Viewer_ sidebar has two panels. The **Viewer Manager** is about the running setup — your Python environment, the viewer, the Jupyter pieces. The **Library Manager** is about what is installed in that environment. Between them, everything the extension manages is visible and one click away.

## The Viewer Manager

Before anything is set up, the panel is your starting point: a **Select Python Interpreter** link, the two **Quickstart** buttons ([Installation](installation.md) walks through them), and a link to the OCP CAD Viewer preferences.

Once set up, it shows the state of your session:

![Viewer Manager](../../assets/viewer-manager.png)

- the **ocp_vscode** entry with the installed version — the top-level health check: if extension and library disagree on major.minor, this is where it shows
- one entry per **running viewer with its port** — the port that `set_port(...)` wants, and the quickest place to look it up when a `show` cannot connect (see [Addressing a viewer](addressing.md))
- the **jupyter / ipython** entries, with buttons to open the viewer and a [Jupyter Console](jupyter_console.md) attached to your kernel

Hovering over the "VIEWER MANAGER" heading reveals a **gear button** that opens the OCP CAD Viewer settings — the fastest route to the [Workspace Config](workspace_config.md).

## The Library Manager

The Library Manager lists the Python libraries the extension cares about — _build123d_, _cadquery_, _ipykernel_, _ocp_tessellate_, _ocp_vscode_ — each with its installed version in the currently selected environment, rescanned when the environment changes (or on demand via the "Refresh libraries list" command).

Per library, hovering reveals the actions:

- the **down-arrow** installs or upgrades the library into the selected environment; VS Code prompts to confirm the environment first, and the command's output lands in the terminal
- the **paste button** inserts the library's import snippet at the cursor — the fastest way to start a file correctly
- **examples (download)** — where a library offers examples, the green download button fetches them into a `<library>_examples` folder next to your work (a file must be open so the extension knows the working folder)

### Customizing the install commands

The commands the Library Manager runs come from two settings, `OcpCadViewer.advanced.quickstartCommands` and `OcpCadViewer.advanced.installCommands`:

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
