# Export

## A viewer as a standalone HTML page

`export_html(filename="cadquery.html", title="CadQuery", viewer=None)` writes the current view — the model, the camera, the tree state and every viewer setting — into one HTML file that opens in any browser without Jupyter or Python:

```python
show(part, viewer="CAD")
# rotate, clip, hide parts, switch tabs ...
export_html("part.html")
```

`viewer=` addresses the viewer [the usual way](addressing.md#addressing-a-viewer): a named sidecar, else the default sidecar, else the viewer the last `show` produced. A sidecar or window is exported as a cell viewer of the same size — the page has no JupyterLab to dock into.

The page carries the model but loads the viewer's JavaScript from the npm registry — `cad-viewer-widget` at exactly the version installed when the export was made — so it needs an internet connection when opened, and only versions that were published render. An export made from a development install of an unpublished version shows an empty page; re-export from a kernel running a released version.

## A notebook as HTML

`jupyter nbconvert --to html notebook.ipynb` keeps the viewers of the cells, provided the notebook carries its widget state:

- either run the notebook interactively, enable *Settings → Save Widget State Automatically* in JupyterLab, and save it before converting,
- or convert with `--execute`, which runs the notebook and saves the state as part of the conversion.

Sidecars and windows are not cell outputs and do not appear in a converted notebook; show into a cell what the HTML should contain.

## PNG

A screenshot of the current view is the shared [`save_screenshot`](../../api.md#screenshots), which works for cell viewers and sidecars alike. The pin button of a cell viewer (see [pinning](addressing.md#size)) does the same interactively and puts the PNG into the cell in place of the live viewer.
