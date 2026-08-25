# Addressing a viewer

A sidecar is named, not dialled: the `viewer=` keyword — accepted by every `show*` command and by the config functions (`status`, `workspace_config`, `combined_config`, `get_default(s)`, `reset_defaults`, `set_viewer_config`) — takes the title the sidecar was opened with:

```python
open_viewer(title="Left")
show(part, viewer="Left")
```

- Showing into a title that does not exist yet opens that sidecar.
- Without `viewer=`, the default sidecar is used (the last one opened with `default=True`).
- `anchor=` says where a newly opened sidecar goes ("right", "split-right", "split-left", "split-top", "split-bottom"); it cannot be changed once the sidecar exists.
