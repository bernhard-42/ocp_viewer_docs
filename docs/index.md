# The OCP CAD viewer documentation

Four viewers are built on [ocp-viewer-core](https://github.com/bernhard-42/ocp-viewer-core): [ocp_vscode](https://github.com/bernhard-42/vscode-ocp-cad-viewer) (the VS Code extension), [ocp_viewer](https://github.com/bernhard-42/ocp-viewer) (the standalone viewer), [Jupyter CadQuery](https://github.com/bernhard-42/jupyter-cadquery) (for Jupyter Lab) and [build123d Studio](https://github.com/bernhard-42/build123d-studio). They all share the same show commands, the same configuration semantics and the same viewer window, because all of that lives in the core. What differs per viewer is only how it is installed and started, where its settings are stored, and how Python reaches it.

Most of these pages document the shared part, valid for every viewer. The per-viewer rest — settings storage, ports and transports, debugging integrations — lives in the [Hosts](#host-specifics-per-viewer) chapter; only installation and quickstart stay with each viewer's own README.

## One convention

Everything a user calls is imported from the host package — the package of the viewer in use. Every name shown on these pages — `show`, `set_defaults`, `Camera`, all of them — is exported identically by all four packages, so every example comes as one tab per viewer:

=== "ocp_viewer"

    ```python
    from ocp_viewer import show
    ```

=== "ocp_vscode"

    ```python
    from ocp_vscode import show
    ```

=== "jupyter_cadquery"

    ```python
    from jupyter_cadquery import show
    ```

=== "build123d_studio"

    ```python
    from build123d_studio import show
    ```

Pick your viewer's tab once — the choice applies to every example on every page and is remembered across visits. The import line is the only thing that names your viewer; the behavior after it is the same everywhere.

The one exception is a handful of keywords that address the viewer's surface rather than its content — `port` (ocp_vscode, ocp_viewer), `viewer` and its sidecar companions `anchor`, `cad_width`, `height`, `pinning` (jupyter_cadquery). Which viewer accepts which is spelled out in [show — Host keywords](show.md#host-keywords).

## The viewer window

- [The viewer](viewer.md) — what is on the screen and how to operate it: mouse navigation, the navigation tree, the toolbar, the tabs, keyboard shortcuts. This is the JavaScript half of the core, embedded by every host.

## Showing objects

- [show](show.md) — show one or more CAD objects, and the full keyword reference shared by all show commands
- [show_object](show_object.md) — show objects incrementally, one call per object
- [push_object / show_objects](push_object.md) — collect objects without rendering, then render them in one batch
- [show_all](show_all.md) — show every CAD object in the current Python scope
- [Additional functions](api.md) — `show_clear`, `save_screenshot`, defaults, viewer state inspection

## Configuring the viewer

- [The config system](config.md) — the three levels of configuration and their precedence, and the `reset_camera` semantics
- [set_viewer_config](set_viewer_config.md) — change a running viewer immediately, without a new show
- [Enums](enums.md) — `Camera`, `Collapse`, `Render`, `AnalysisTool`, `UiTab` and the Studio enums

## Analysis tools

- [Measure mode](measure.md) — properties and distance/angle measurement in the viewer
- [Object selection](selector.md) — pick faces, edges or vertices in the viewer and use their indices in code

## Appearance and scene

- [Color maps](colormaps.md) — automatic color assignment for object collections
- [Materials and Studio mode](pbr_studio.md) — PBR materials and photo-realistic rendering
- [ImageFace](image_face.md) — place a 2-D image as a reference plane in the scene
- [Animation](animation.md) — drive keyframe animations on shown assemblies

## Host specifics, per viewer

One chapter per viewer for what genuinely differs: settings storage, port discovery and `set_port`, sidecars, visual debugging integrations, editor tooling, troubleshooting. Installation and quickstart stay in each viewer's own README — that is what its marketplace and package pages render.

- [ocp_vscode](hosts/ocp_vscode.md)
- [ocp_viewer](hosts/ocp_viewer.md)
- [jupyter_cadquery](hosts/jupyter_cadquery.md)
- [build123d_studio](hosts/build123d_studio.md)
