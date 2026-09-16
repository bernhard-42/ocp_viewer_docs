# Addressing a viewer

Several OCP Viewer servers can run at once, each on its own port. The `port=` keyword — accepted by every `show*` command and by the config functions (`status`, `workspace_config`, `combined_config`, `get_default(s)`, `reset_defaults`, `set_viewer_config`, `save_screenshot`) — says which one a call means.

- `set_port(port)` — pin the port once for the Python process instead of passing `port=` per call (the recommended way when several viewers run)
- `get_port()` — the currently selected port, triggering discovery on first use
- `find_and_set_port()` — re-run the discovery: read the running viewers from `~/.ocpvscode`, probe them, and prompt if more than one is alive
- `OCP_PORT` env var — overrides discovery entirely, useful in CI or scripts

With a single viewer running, none of this is needed — discovery picks it up automatically.

## The registry `~/.ocpvscode`

Discovery works through one file shared by every viewer of the family: `~/.ocpvscode`. Each OCP Viewer server writes its port there when it starts and removes it when it stops, the VS Code extension does the same for its viewers, and every `show()` — from any of the packages — reads the list to find a running viewer. That is what lets a script address an OCP Viewer and an OCP CAD Viewer for VS Code interchangeably on one machine. **OCP Viewer therefore creates and maintains `~/.ocpvscode` even when neither VS Code nor the ocp_vscode extension is installed**; the name is historical, and it stays so that existing installations keep working.

The file records one more thing: when `show()` runs inside a Jupyter kernel (JupyterLab, a notebook, an editor REPL), the client stores the kernel's connection file next to the port it found, and says so:

```
Jupyter kernel running
Jupyter connection file path written to <HOME>/.ocpvscode
```

This is what the VS Code extension's *Open Jupyter console* uses to attach a console to the kernel driving a viewer. Nothing in OCP Viewer reads it; the lines are printed so that a write to a file in your home directory never happens silently. A viewer that did not shut down cleanly leaves its port in the file — harmless, discovery probes every entry and offers only the live ones. If the file is missing, it is created on first use; deleting it while no viewer runs is safe.
