# Additional Python API

Functions exported by every viewer's package that aren't covered by the dedicated `show*` pages.

Function signatures on this page and on [the config system](config.md) page show only the shared parameters. Where a function needs to say _which_ viewer it means, the keyword differs per host — see "Addressing a viewer" in your viewer's chapter: [VS Code CAD Viewer](hosts/ocp_vscode/addressing.md), [OCP Viewer](hosts/ocp_viewer/addressing.md), [Jupyter CadQuery](hosts/jupyter_cadquery/addressing.md). build123d Studio has exactly one viewer and no addressing keyword.

## Screenshots

- `save_screenshot(filename, polling=True, progress_only=False)`

  Save the current viewer view to a PNG. `filename` is taken relative to the current working directory unless it's absolute. With `polling=True` the call returns only after the file has been written; this is what `Animation.save_as_gif` relies on internally.

## Clear viewer

- `show_clear()`

  Clear the viewer (remove all currently shown objects, reset the navigation tree). The viewer shows the splash logo again until the next `show*` call.

## Blueprint images

- `ImageFace`

  Re-exported from `ocp_tessellate.cad_objects`. Lets you place a 2-D image as a face in the scene — e.g. a photo or technical drawing to model against. See [image_face.md](image_face.md).
