# Troubleshooting

## Is everything installed?

Jupyter CadQuery is two extensions: the JupyterLab extension that draws (shipped inside cad-viewer-widget) and a Jupyter Server extension that runs the [measurement backend](concepts.md#where-the-backend-runs-and-how-measurements-happen). Each has its own listing:

```bash
jupyter lab extension list        # must show:  jupyter_cadquery <version> OK
jupyter server extension list     # must show:  jupyter_cadquery <version> OK
```

A line `Extension package jupyter_cadquery took N s to import` in the second listing is not a problem: it is OCP and VTK loading, which the check imports. On a Mac the very first run can take minutes for the same reason.

## A cell stays at `[*]` and the kernel is idle

ipykernel 7.x can stall a running notebook: a cell stops at `[*]` while the kernel sits idle, most often during *Run All Cells*. It is [an ipykernel defect](https://github.com/ipython/ipykernel/pull/1529), not the viewer's — it happens with no viewer involved at all — and it is fixed upstream but not released as of ipykernel 7.3.

- Until the release that carries the fix: `pip install "ipykernel<7"` (6.31.0 is fine).
- When it happens: *Kernel → Reconnect to Kernel* resumes the run exactly where it stopped, without restarting the kernel and without losing state.

## After an upgrade nothing changed, or the viewer errors

The JupyterLab extension is loaded when the page loads. After `pip install -U jupyter-cadquery` (or cad-viewer-widget) reload the browser page — restarting the kernel is not enough, it keeps the old JavaScript. A version mismatch between the two halves shows in the browser console as a failed model or view lookup.

## Measurements do nothing

The analysis tools ask the server extension over HTTP. Two things break that:

- on **binder** its security restrictions block the request — the viewer works, measuring does not;
- the server extension is not installed or not enabled — check the `jupyter server extension list` above.

## An exported HTML page is empty

The page loads `cad-viewer-widget` from the npm registry at the version installed when the export was made. Only published versions exist there: an export from a development or pre-release install shows an empty page and a *Could not create a view* in the browser console. Re-export from a kernel running a released version — see [Export](export.md).

## `show` draws into the wrong place

`show` goes to the default sidecar if one is open, else into the cell. `get_default_viewer()` says which sidecar is the default, `close_viewers()` gets you back to cell viewers, and `viewer="<title>"` addresses one explicitly — see [Sidecars, windows and cells](addressing.md).
