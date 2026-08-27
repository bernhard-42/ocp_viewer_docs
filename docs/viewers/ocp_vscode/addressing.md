# Addressing a viewer

VS Code can run several instances and each VS Code instance can run its own CAD Viewer, each listening on its own port. The `port=` keyword — accepted by every `show*` command and by the config functions (`status`, `workspace_config`, `combined_config`, `get_default(s)`, `reset_defaults`, `set_viewer_config`, `save_screenshot`) — says which one a call means.

- `set_port(port)` — pin the port once for the Python process instead of passing `port=` per call (the recommended way when several viewers run)
- `get_port()` — the currently selected port, triggering discovery on first use
- `find_and_set_port()` — re-run the discovery: read the running viewers from `~/.ocpvscode`, probe them, and prompt if more than one is alive
- `OCP_PORT` env var — overrides discovery entirely, useful in CI or scripts

With a single viewer running, none of this is needed — discovery picks it up automatically.

## The registry `~/.ocpvscode`

Every running viewer (this one and the standalone [OCP Viewer](../ocp_viewer/addressing.md)) registers its port in `~/.ocpvscode`:

```json
{
    "version": 2,
    "services": {
        "3939": "<jupyter connection file or empty>",
        "3940": ""
    }
}
```

Maintained automatically — you should not need to edit it. When a viewer shuts down cleanly its entry is removed, and discovery probes each listed port with a 1-second TCP check anyway, so a stale entry from a crashed viewer is harmless. On the first `show*` call in a Python process, discovery reads this file, drops dead ports, uses a single live one silently, and prompts when several are alive (a `questionary` list in a terminal; a VS Code input box in a Jupyter kernel).

## Status bar

The extension shows the active viewer's port in the status bar: `OCP: <port>` (viewer running) or `OCP: <port>·DEBUG` (visual debugging on — see [Visual debugging](visual_debugging.md)).

## Common situations

| Situation                                   | What to do                                                                                           |
| ------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| Only one viewer running                     | Nothing — discovery picks it automatically                                                           |
| Multiple viewers, same project              | `set_port(<port>)` at the top of the script, or pick from the prompt on first `show`                 |
| Running tests / CI                          | `OCP_PORT=<port>` in the environment                                                                 |
| Stale entry in `~/.ocpvscode` after a crash | To clean up by hand, edit the JSON                                                                   |
| "Cannot access viewer config"               | The Python side picked a port nothing is listening on — run `find_and_set_port()` or `set_port(...)` |

## Troubleshooting

If `~/.ocpvscode` is out of sync with reality — viewers listed that are long gone, a running viewer missing, discovery behaving strangely — replace its content with the empty registry:

```json
{
    "version": 2,
    "services": {}
}
```

It is rebuilt automatically: every viewer registers itself again on its next start.
