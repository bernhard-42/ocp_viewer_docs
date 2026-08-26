# Versioning and compatibility

The shared core is one project in two halves — `ocp-viewer-core` on PyPI (the Python half) and `ocp-viewer-core` on npm (the JavaScript half) — and the version scheme says one thing about the wire between them:

**major.minor names the contract and must match between the halves; the patch level is each half's own.**

| Python | JavaScript | valid? |
| ------ | ---------- | ------ |
| 4.1.2 | 4.1.2 | yes |
| 4.1.2 | 4.1.5 | yes — a JS-only fix shipped |
| 4.1.5 | 4.1.2 | yes — a Python-only fix shipped |
| 4.1.2 | 4.2.0 | **no** |
| 4.2.0 | 4.1.2 | **no** |

Everything that crosses the boundary — the protocol, the config vocabulary, the message shapes — only changes with a minor (or major) bump, always on both halves. A patch touches one side's internals alone, which is the point: a pure Python fix reaches you with `pip install -U` and no viewer needs a release for it.

## How the viewers consume the core

- The Python side of every viewer declares a **minor range** (e.g. `>=1.1.0, <1.2.0`): any patch of that minor works; the next minor is opted into deliberately.
- The JavaScript side is bundled *into* each viewer at build time — the VS Code extension ships it inside the `.vsix`, OCP Viewer serves its copy as static files, Jupyter CadQuery compiles it into the labextension, Studio into the app. A JS-only patch is picked up by whichever viewer rebuilds for it.

## The handshake

The contract is enforced at runtime, not just promised: the Python half sends its version with every model, and the CAD Viewer page compares major.minor against its own JavaScript version — on a mismatch it raises a loud console error and reports into the viewer's log. So a drifted pairing announces itself instead of showing up as the vaguest of symptoms, an option that silently does nothing.

## The viewer pairing

Separately from the core's two halves, each viewer's own frontend and Python package must also agree: the VS Code extension checks itself against the `ocp_vscode` library at major.minor and reports it (a patch-level difference is fine); the other viewers ship both halves in one artifact and cannot drift. A mismatch here is the first thing [troubleshooting](../viewers/ocp_vscode/troubleshooting.md) asks about.
