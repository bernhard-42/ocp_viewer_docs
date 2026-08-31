# The OCP CAD viewer documentation

There are four viewers in the OCP Viewer ecosystem:

- [ocp_vscode](https://github.com/bernhard-42/vscode-ocp-cad-viewer) (the VS Code extension)
- [ocp_viewer](https://github.com/bernhard-42/ocp-viewer) (the standalone viewer)
- [Jupyter CadQuery](https://github.com/bernhard-42/jupyter-cadquery) (for Jupyter Lab)
- [build123d Studio](https://github.com/bernhard-42/build123d-studio) (the standalone application with an integrated python environment)

![](./assets/ocp_vscode.png#only-light){width=48%}
![](./assets/ocp_viewer.png#only-light){width=48%}
![](./assets/jupyter-cadquery.png#only-light){width=48%}
![](./assets/build123d-studio.png#only-light){width=48%}

![](./assets/ocp_vscode-dark.png#only-dark){width=48%}
![](./assets/ocp_viewer-dark.png#only-dark){width=48%}
![](./assets/jupyter-cadquery-dark.png#only-dark){width=48%}
![](./assets/build123d-studio-dark.png#only-dark){width=48%}

///caption
VS Code CAD Viewer | OCP Viewer <br> Jupyter Cadquery | Build123d Studio
///

They are all built on [ocp-viewer-core](https://github.com/bernhard-42/ocp-viewer-core), and share the same show commands, the same configuration semantics and the same viewer window, see [Architecture](concepts/architecture.md). What differs per viewer is only how it is installed and started, where its settings are stored, and how Python reaches it.

This documentation first introduces the four viewers in _[Viewers](#viewers)_: how they are installed, how to run code, where settings are stored, and their viewer-specific features.

The chapters _[CAD Viewer](#the-cad-viewer)_ and _[The Python API](#the-python-api)_ then document the shared part, valid for every viewer.

The underlying concepts, e.g. how the pieces fit together, are explained in the _[Concepts](#concepts)_ chapter.

!!! info "One vocabulary, two import styles"

    Every name shown on these pages — `show`, `set_defaults`, `Camera`, all of them — is exported identically by all four viewer packages. The examples use the portable form, which resolves to whichever viewer your environment runs:

    ```python
    from ocp_viewer_core.viewer import show
    ```

    Importing from your viewer's own package works exactly the same — `from ocp_vscode import show` in VS Code, `from ocp_viewer import show` for OCP Viewer, and so on. The import line is the only thing that names your viewer; the behavior after it is the same everywhere. [Importing](importing.md) explains both styles.

    The one exception is a handful of keywords that address the viewer's surface rather than its content — `port` (ocp_vscode, ocp_viewer), `viewer` and its sidecar companions `anchor`, `cad_width`, `height`, `pinning` (jupyter_cadquery). Which viewer accepts which is spelled out in [show — Viewer-specific keywords](show.md#viewer-specific-keywords).

## Viewers

One chapter per viewer for what genuinely differs: installation, settings storage, port discovery and `set_port`, sidecars, visual debugging integrations, editor tooling, troubleshooting.

- VS Code CAD Viewer — [Installation](viewers/ocp_vscode/installation.md), [Viewer and Library Manager](viewers/ocp_vscode/managers.md), [Workspace Config](viewers/ocp_vscode/workspace_config.md), [Addressing a viewer](viewers/ocp_vscode/addressing.md), [Visual debugging](viewers/ocp_vscode/visual_debugging.md), [Jupyter Console](viewers/ocp_vscode/jupyter_console.md), [Commands and snippets](viewers/ocp_vscode/commands.md), [Troubleshooting](viewers/ocp_vscode/troubleshooting.md), [Concepts](viewers/ocp_vscode/concepts.md)
- OCP Viewer — [Installation](viewers/ocp_viewer/installation.md), [Workspace Config](viewers/ocp_viewer/workspace_config.md), [Addressing a viewer](viewers/ocp_viewer/addressing.md), [Editor support: NeoVim](viewers/ocp_viewer/neovim.md), [Docker](viewers/ocp_viewer/docker.md), [Concepts](viewers/ocp_viewer/concepts.md)
- Jupyter CadQuery — [Installation](viewers/jupyter_cadquery/installation.md), [Workspace Config](viewers/jupyter_cadquery/workspace_config.md), [Addressing a viewer](viewers/jupyter_cadquery/addressing.md), [Concepts](viewers/jupyter_cadquery/concepts.md)
- build123d Studio — [Installation](viewers/build123d_studio/installation.md), [First Run](viewers/build123d_studio/first_run.md), [The window](viewers/build123d_studio/window.md), [Running code](viewers/build123d_studio/running.md), [Debugging](viewers/build123d_studio/debugging.md), [Console and variables](viewers/build123d_studio/console_variables.md), [The editor](viewers/build123d_studio/editor.md), [Packages and the environment](viewers/build123d_studio/packages.md), [Workspace Config](viewers/build123d_studio/workspace_config.md), [Commands and shortcuts](viewers/build123d_studio/commands.md), [Troubleshooting](viewers/build123d_studio/troubleshooting.md), [Concepts](viewers/build123d_studio/concepts.md)

## The CAD Viewer

- [Overview](viewer.md) — the window's layout, the toolbar, the info box, and what the viewer remembers. This is the JavaScript half of the core, embedded by every viewer.
- [Mouse and Keys](mouse_keys.md) — navigation and selection bindings, and remapping the modifier keys
- [Tabs](tabs.md) — the tree panel's five tabs: Tree, Clip, Zebra, Material, Studio
- Analysis tools:
  - [Overview](measure_select.md) — picking and filters, programmatic activation, mesh-based vs CAD-exact numbers
  - [Measure mode](measure.md) — the Distance and Properties tools and their panels
  - [Object selection](selector.md) — pick faces, edges or vertices and use their indices in code
- [Dynamic features](dynamic_features.md) — explode and the animation bar

## The Python API

**Showing objects:**

- [show](show.md) — show one or more CAD objects, and the full keyword reference shared by all show commands
- [show_object](show_object.md) — show objects incrementally, one call per object
- [push_object / show_objects](push_object.md) — collect objects without rendering, then render them in one batch
- [show_all](show_all.md) — show every CAD object in the current Python scope
- [Keeping the camera orientation](reset_camera.md) — the `reset_camera` semantics: keep, recenter, reset, or snap to a preset
- [Additional functions](api.md) — `show_clear`, `save_screenshot`, and blueprint images

**Configuring:**

- [The config system](config.md) — the three levels of configuration and their precedence, and the defaults and state inspection functions
- [set_viewer_config](set_viewer_config.md) — change a running viewer immediately, without a new show
- [Enums](enums.md) — `Camera`, `Collapse`, `Render`, `AnalysisTool`, `UiTab` and the Studio enums

**Appearance:**

- [Color maps](colormaps.md) — automatic color assignment for object collections
- [Materials and Studio mode](pbr_studio.md) — PBR materials and photo-realistic rendering
- [ImageFace](image_face.md) — place a 2-D image as a reference plane in the scene

**Animation:**

- [Animation](animation.md) — drive keyframe animations on shown assemblies

## Concepts

- [Architecture](concepts/architecture.md) — the ecosystem's pieces and what happens when you call `show()`
- [Python to CAD Viewer communication](concepts/communication.md) — the transport contract, and the four ways a model travels
- [The measurement backend](concepts/backend.md) — where the CAD-exact numbers come from
- [Configuration layers](concepts/configuration.md) — settings, defaults, keywords and the viewer's own state
- [Mesh creation](concepts/tessellation.md) — how tessellation works, and what keeps it fast
- [Versioning and compatibility](concepts/versioning.md) — one core version, two registries, and why versions must match

## Versions

This documentation describes the following versions — patch releases within each line stay covered:

| Component                                                                                         | Documented version |
| ------------------------------------------------------------------------------------------------- | ------------------ |
| [ocp-viewer-core](https://github.com/bernhard-42/ocp-viewer-core)                                 | 1.0.x              |
| [OCP CAD Viewer for VS Code](https://github.com/bernhard-42/vscode-ocp-cad-viewer) (`ocp_vscode`) | 4.1.x              |
| [OCP Viewer](https://github.com/bernhard-42/ocp-viewer) (`ocp_viewer`)                            | 1.0.x              |
| [Jupyter CadQuery](https://github.com/bernhard-42/jupyter-cadquery) (`jupyter_cadquery`)          | 5.1.x              |
| [cad-viewer-widget](https://github.com/bernhard-42/cad-viewer-widget)                             | 4.1.x              |
| [build123d Studio](https://github.com/bernhard-42/build123d-studio)                               | 0.4.x              |
| [three-cad-viewer](https://github.com/bernhard-42/three-cad-viewer)                               | 5.0.x              |
| [ocp-tessellate](https://github.com/bernhard-42/ocp-tessellate)                                   | 3.5.x              |
