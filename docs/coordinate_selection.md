# Coordinate selection

With the mouse over the model, `Cmd-C` (macOS) or `Ctrl-C` (Windows, Linux) copies the coordinates of the point under the cursor to the clipboard, as `x, y, z` with four decimals:

```text
12.3457, 0.0000, -7.5000
```

The status line at the bottom of the canvas confirms it with `copied: …`. The coordinates are world coordinates of the visible surface where the cursor touches it, ready to be pasted into code, e.g. `Pos(12.3457, 0.0000, -7.5000)`. No toolbar tool needs to be active.

`Cmd-C` / `Ctrl-C` copies nothing — and keeps its normal meaning — when

- no visible part is under the cursor,
- explode mode is on, an animation has been started (Play or the time slider), or the z-scale is not 1, because parts are then not at their model positions,
- a text field has the focus or text is selected.

!!! warning "VS Code: click into the viewer first"

    In VS Code the viewer is a separate frame that receives the keyboard only after a click into it. Hovering over it is not enough: while the editor has the focus, `Cmd-C` / `Ctrl-C` goes to the editor. Click into the canvas once; from then on `Cmd-C` / `Ctrl-C` copies the point under the cursor.

!!! warning "Clipping cap faces"

    Copying the coordinates only works on the original faces/edges/vertices, not on faces/edges/vertices created by the clipping planes.
