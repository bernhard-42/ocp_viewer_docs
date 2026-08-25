# Addressing a viewer

VS Code can run several viewer panels, each listening on its own port. The `port=` keyword — accepted by every `show*` command and by the config functions (`status`, `workspace_config`, `combined_config`, `get_default(s)`, `reset_defaults`, `set_viewer_config`, `save_screenshot`) — says which one a call means.

- `set_port(port)` — pin the port once for the Python process instead of passing `port=` per call (the recommended way when several viewers run)
- `get_port()` — the currently selected port, triggering discovery on first use
- `find_and_set_port()` — re-run the discovery: read the running viewers from `~/.ocpvscode`, probe them, and prompt if more than one is alive
- `OCP_PORT` env var — overrides discovery entirely, useful in CI or scripts

With a single viewer running, none of this is needed — discovery picks it up automatically.
