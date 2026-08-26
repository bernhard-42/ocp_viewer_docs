# Concepts

How the shared pieces are wired in Studio. The general mechanisms are in the [Concepts](../../concepts/architecture.md) chapter; this page is build123d Studio's shape of them.

## The integration chain

```text
Jupyter kernel ──ws──▶ sidecar ──ipc──▶ app window (webview)
    show()              (the hub)         ocp-viewer-core JS page
                                            └─▶ three-cad-viewer
```

Four processes: the **app** (a Neutralino window whose webview runs the editor and the core's JavaScript page with three-cad-viewer), the **sidecar** (the message hub the app talks to), the **Jupyter kernel** your code runs on, and the **measurement process**. Studio brings all of them up itself, inside its own self-built Python environment.

## Python to CAD Viewer

`show()` in your code (on the kernel, or in a Run-File process that was given the viewer's address) sends the model to the sidecar over a WebSocket; the sidecar forwards it over IPC into the app's webview, where the shared page renders it. Models ride a dedicated binary channel so their buffers arrive without copying, and `show_clear` rides the same channel so it stays ordered against the models it clears. The viewer's state flows back the same way, which is how `status()` answers. There is exactly one viewer per window — no addressing keyword exists.

## Where the backend runs and how measurements happen

The [measurement backend](../../concepts/backend.md) runs as its **own measurement process** beside the kernel — deliberately not on the kernel, so a measurement never competes with your running code. The sidecar keeps the id-to-shape mapping of the last show and feeds it to that process; when you pick shapes with an analysis tool, the webview's notification goes to the sidecar, the measurement process computes the exact answer from the BRep, and the sidecar delivers it back into the page.
