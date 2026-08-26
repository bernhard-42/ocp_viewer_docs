# ocp_viewer_docs

The documentation of the OCP CAD viewer family — [ocp_vscode](https://github.com/bernhard-42/vscode-ocp-cad-viewer), [ocp_viewer](https://github.com/bernhard-42/ocp-viewer), [Jupyter CadQuery](https://github.com/bernhard-42/jupyter-cadquery) and [build123d Studio](https://github.com/bernhard-42/build123d-studio) — in one place: the shared show commands, the config system, the viewer window, and one chapter per viewer for its specifics.

Published site: **https://bernhard-42.github.io/ocp_viewer_docs/**

This repo contains no code. The behavior it documents lives in [ocp-viewer-core](https://github.com/bernhard-42/ocp-viewer-core) and [three-cad-viewer](https://github.com/bernhard-42/three-cad-viewer); the viewers add only their transports and settings.

## Building locally

The site is built with [Zensical](https://zensical.org/), the successor of Material for MkDocs by the same team, reading the unchanged `mkdocs.yml`:

```bash
pip install -r requirements.txt
zensical serve          # live preview at http://127.0.0.1:8000
zensical build --strict # what CI runs
```

Every push to `main` builds the site and deploys it to GitHub Pages via the workflow in `.github/workflows/docs.yml`.
