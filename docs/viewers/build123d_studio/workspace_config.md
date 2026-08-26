# Workspace Config

Studio's viewer settings live in **Settings → Viewer**, reached from the toolbar's Settings button. They are stored in the application's `settings.json` in the [data directory](first_run.md#the-first-start) and form the viewer-settings tier of the [config system](../../config.md): the starting values that `set_defaults()` overrides per Python process and show keywords per call.

The tab groups them as:

- **Layout and tree** — glass mode, toolbar, tree width, initial tree collapse, new tree behaviour (eye icon controls the whole object, mesh icon only the wireframe — see [Tree](../../tabs.md#tree))
- **Camera and controls** — camera behavior on a new model (`reset_camera`, default `KEEP` — see [Keeping the camera orientation](../../reset_camera.md)), orthographic camera, up axis, orbit vs trackball control, rotate/zoom/pan speeds, and the modifier keys for the [mouse bindings](../../mouse_keys.md#modifier-keys)
- **Grid and axes** — axes, axes at the origin, the XY/XZ/YZ grid planes, grid centering, grid font size, ticks
- **Objects** — black edges, transparency and its opacity, explode
- **Rendering** — deviation and angular tolerance (tessellation), ambient/direct intensity, metalness, roughness, and the default colours for shapes, edges, thick edges, faces and vertices

The defaults match the other viewers key for key — a value learnt in one viewer means the same in the next.

Beyond the Viewer tab, Settings also holds Studio's own concerns — packages (which build123d and ocp-viewer-core to install: PyPI, dev branch or local checkout), the editor, the new-file template, debugging, the keyboard shortcuts, and the debug-console log level. Those are Studio's application settings, not viewer configuration, and the [repository documentation](https://github.com/bernhard-42/build123d-studio#readme) covers them.
