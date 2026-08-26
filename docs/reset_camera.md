# Keeping the camera orientation

Sometimes it is helpful to keep the orientation of an object across code changes. This is what the `reset_camera` keyword of every `show*` command does:

- `reset_camera=Camera.CENTER` will keep position and rotation, but ignore panning. This means the new object will be repositioned to the center (most robust approach).
- `reset_camera=Camera.KEEP` will keep position, rotation and panning.
- `reset_camera=Camera.RESET` will ensure that position, rotation and panning will be reset to the initial default.
- `reset_camera=Camera.ISO` / `Camera.TOP` / `Camera.BOTTOM` / `Camera.LEFT` / `Camera.RIGHT` / `Camera.FRONT` / `Camera.BACK` snaps to one of the axis-aligned camera presets — useful for reproducible screenshots.

!!! warning

    The default is `Camera.KEEP`, so between show calls the camera position is stable.

    However, panning can be problematic. When the next object to be shown is much larger or smaller and the object before was panned, it can happen that nothing is visible (the new object at the pan location is outside of the viewer frustum). Use the view buttons to get the object back into the viewer.

    `show` warns when this may have happened ("Object may be too small to see", "may extend beyond view", "may be outside visible area"); silence the warnings with `ignore_camera_warnings()`.

See [Camera](enums.md#camera) for the enum reference.
