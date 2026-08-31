# Color maps

A color map automatically assigns colors to the objects passed to `show()` / `show_all()`. Useful for collections (lists/tuples) where you don't want to spell out a color for each entry.

```python
from ocp_viewer_core.viewer import show, set_colormap, ColorMap
```

## Activation

### Use one of the named maps

```python
from build123d import Sphere, Pos
set_colormap(ColorMap.tab20())

show(*[Pos(i * 3) * Sphere(1) for i in range(20)])
```

![](./assets/colormap-tab20.png#only-light){.center width=32%}
![](./assets/colormap-tab20-dark.png#only-dark){.center width=32%}

### Use a generator-style endless palette

```python
from build123d import Sphere, Pos
set_colormap(ColorMap.golden_ratio())

show(*[Pos(i * 3) * Sphere(1) for i in range(20)])
```

![](./assets/colormap-golden-ratio.png#only-light){.center width=32%}
![](./assets/colormap-golden-ratio-dark.png#only-dark){.center width=32%}

### Unset the color map

```python
unset_colormap()
# or
set_colormap(None)
```

### Get the currently set color map

`get_colormap()` returns the currently active map (resetting its iterator).

!!! info

    The map is consumed each time in `show`, so order matters — the n-th object gets the n-th color.

## Available factories

`ColorMap` exposes the following static factory methods:

### Listed palettes (cycling)

| Factory                                      | Source                           |
| -------------------------------------------- | -------------------------------- |
| `ColorMap.accent(alpha=1.0, reverse=False)`  | matplotlib's _Accent_ (8 colors) |
| `ColorMap.dark2(alpha=1.0, reverse=False)`   | _Dark2_ (8)                      |
| `ColorMap.paired(alpha=1.0, reverse=False)`  | _Paired_ (12)                    |
| `ColorMap.pastel1(alpha=1.0, reverse=False)` | _Pastel1_ (9)                    |
| `ColorMap.pastel2(alpha=1.0, reverse=False)` | _Pastel2_ (8)                    |
| `ColorMap.set1(alpha=1.0, reverse=False)`    | _Set1_ (9)                       |
| `ColorMap.set2(alpha=1.0, reverse=False)`    | _Set2_ (8)                       |
| `ColorMap.set3(alpha=1.0, reverse=False)`    | _Set3_ (12)                      |
| `ColorMap.tab10(alpha=1.0, reverse=False)`   | _tab10_ (10)                     |
| `ColorMap.tab20(alpha=1.0, reverse=False)`   | _tab20_ (20)                     |
| `ColorMap.tab20b(alpha=1.0, reverse=False)`  | _tab20b_ (20)                    |
| `ColorMap.tab20c(alpha=1.0, reverse=False)`  | _tab20c_ (20)                    |

### Procedural palettes

| Factory                                                                                    | Description                                                                                |
| ------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------ |
| `ColorMap.golden_ratio(colormap="hsv", alpha=1.0, reverse=False)`                          | Walks the colormap by the inverse golden ratio so successive colors are maximally distinct |
| `ColorMap.seeded(seed_value=42, colormap="hsv", alpha=1.0, **params)`                      | Deterministic random walk through the colormap (use `colormap="rgb"` to draw RGB directly) |
| `ColorMap.segmented(length=10, colormap="hsv", alpha=1.0, reverse=False)`                  | Equally spaced samples across a continuous colormap                                        |
| `ColorMap.listed(length=10, colormap="mpl:plasma", colors=None, alpha=1.0, reverse=False)` | Subsample a matplotlib listed colormap, or pass `colors=[...]` to use your own list        |

```python
golden_ratio = ColorMap.golden_ratio()
seeded = ColorMap.seeded(42, "hsv", alpha=0.8)
segmented = ColorMap.segmented(20, "hsv")
listed = ColorMap.listed(colors=["red", "green", "blue"])
```

!!! warning

    Procedural palettes are endless (generated, starting over at element 0 if the end is reached). So do **not** `list(palette)`. To get the first 10 colors, use `[next(palette) for i in range(10)]`

For matplotlib-based variants pass `colormap="mpl:<name>"` (e.g. `"mpl:plasma"`). The `mpl:` form requires matplotlib to be installed.

```python
colors = set_colormap(ColorMap.listed(20, "mpl:turbo", reverse=False))
```

## Custom maps

Subclass `BaseColorMap` and implement `__next__` returning an `(r, g, b, alpha)` tuple. Use `set_colormap(YourMap())` to activate it.

For a working end-to-end example see [`examples/colormaps.py` in the ocp_vscode repository](https://github.com/bernhard-42/vscode-ocp-cad-viewer/blob/main/examples/colormaps.py).
