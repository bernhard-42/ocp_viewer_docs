# Dynamic features

## Explode

**Explode** (`explode=True`) animates an assembly apart, each part moving away from the center. The animation control bar appears — scrub the timeline slider to any intermediate state. Explode is mutually exclusive with the analysis tools.

![explode](assets/viewer-explode.png#only-light){ .center width="48%" }
![explode-dark](assets/viewer-explode-dark.png#only-dark){ .center width="48%" }

/// caption
Explosion view
///

## The animation bar

A control bar with a timeline slider and play / pause / stop buttons, shown below the canvas whenever there is something to play: an animation sent from Python (see [Animation](animation.md)) or the explode animation. The slider scrubs; from Python, `set_relative_time(fraction)` does the same, which is how `save_as_gif` renders frame by frame.

