# Concepts

How the shared pieces are wired in JupyterLab. The general mechanisms are in the [Concepts](../../concepts/architecture.md) chapter; this page is Jupyter CadQuery's shape of them.

## The integration chain

```text
notebook kernel ──traitlets──▶ cad-viewer-widget ──▶ sidecar / cell output
     show()                     (JupyterLab                └─▶ three-cad-viewer
                                 extension)
```

Jupyter CadQuery is the one viewer without the core's JavaScript page: its frontend is [cad-viewer-widget](https://github.com/bernhard-42/cad-viewer-widget), an ipywidgets widget wrapping three-cad-viewer directly, shipped as a prebuilt JupyterLab extension. The shared building blocks (the core's `apply`, `animate`, notification handling) are compiled into it, so the CAD Viewer still behaves identically.

## Python to CAD Viewer

The transport is the **widget's traitlets** over the Jupyter comm channel — no socket of this viewer's own, no port. A traitlet has the same name on both sides, so this transport skips the camelCase translation the socket viewers do: what Python sets is what the JavaScript reads. Models are sent as binary buffers through the same channel; the two questions a show asks are answered in-process (`config` from `~/.jcq_config`, `status` from the widget's synced state). A viewer is [addressed by title](addressing.md#addressing-a-viewer), not dialled by port.

## Where the backend runs and how measurements happen

The [measurement backend](../../concepts/backend.md) lives in a **Jupyter Server extension** — the one place in a Jupyter deployment that runs Python outside your kernel and is reachable from the browser. After each show, the kernel posts the model's id-to-shape mapping to it over HTTP. When you pick shapes with an analysis tool, the *browser* itself makes an HTTP request to that server extension, the backend computes the exact answer, and the HTTP reply carries it back — landing in the widget as a trait update. (This HTTP hop is also why measurements do not work on binder: its security restrictions block the request.)
