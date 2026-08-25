# Additional Python API

Functions exported by every viewer's package that aren't covered by the dedicated `show*` pages.

## Addressing a viewer

Where a function below needs to say _which_ viewer it means, the keyword differs per host, exactly as on the `show` family (see [host keywords](show.md#host-keywords)):

- `port=` — ocp_vscode and ocp_viewer, to pick one of several open viewers; typically set once with `set_port(port)` instead of per call
- `viewer=` — jupyter_cadquery, naming the sidecar by the title it was opened with (`open_viewer(title=...)`)
- build123d_studio has exactly one viewer, so it adds no addressing keyword

The signatures below show only the shared parameters; your viewer's variants of `status`, `workspace_config`, `combined_config`, `get_default(s)`, `reset_defaults`, `set_viewer_config` and `save_screenshot` additionally accept its addressing keyword.

## Screenshots

- `save_screenshot(filename, polling=True, progress_only=False)`

  Save the current viewer view to a PNG. `filename` is taken relative to the current working directory unless it's absolute. With `polling=True` the call returns only after the file has been written; this is what `Animation.save_as_gif` relies on internally.

## Object stack management

- `show_clear()`

  Clear the viewer (remove all currently shown objects, reset the navigation tree). The viewer shows the splash logo again until the next `show*` call.

- `remove_object(name, call_show=False, progress="-+*c")`

  Remove a single named object from the incremental object stack used by `push_object` / `show_objects`. If `call_show=True`, immediately re-renders the remaining stack.

- `reset_show()`

  Reset the object stack so the next `show_object(..., clear=True)` starts from a clean slate. Convenience around the same internal registry that `push_object` and `show_objects` work on.

## Defaults

- `set_defaults(**kwargs)`

  Persist values across subsequent `show*` calls in this Python process. Accepts the same viewer keywords as `show` (see [show.md](show.md)). The host's settings provide the starting values; `set_defaults` overrides them. See [the config system](config.md) for the full precedence.

- `reset_defaults()`

  Re-apply the host's settings on top of the running viewer and clear the in-process defaults set via `set_defaults`.

- `get_default(key)` / `get_defaults()`

  Read a single default or the full merged default dict (host settings + `set_defaults`).

## Viewer state

- `status(debug=False)`

  Return the live state dict of the viewer (camera position, current tab, visibility states, …).

- `workspace_config()`

  Return the host's configuration as the viewer currently sees it. Raises `RuntimeError` if the viewer isn't reachable.

- `combined_config()`

  `workspace_config` merged with the live `status` and any `set_defaults` overrides — useful for "what will the next `show` actually use?" inspection.

## Imports & utilities

- `ImageFace`

  Re-exported from `ocp_tessellate.cad_objects`. Lets you place a 2-D image as a face in the scene. See [image_face.md](image_face.md).
