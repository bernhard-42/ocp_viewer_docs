# Concepts

How the shared pieces are wired inside VS Code. The general mechanisms are in the [Concepts](../../concepts/architecture.md) chapter; this page is the VS Code CAD Viewer's shape of them.

## The integration chain

```text
your Python process ──ws──▶ VS Code extension ──▶ webview panel
     show()                  (the server in           ocp-viewer-core JS page
                              the middle)                └─▶ three-cad-viewer
```

Three processes and a webview: your Python process runs the core's show pipeline; the **extension host** runs a WebSocket server; the **webview panel** runs the core's JavaScript page, which embeds three-cad-viewer. The extension is the relay in the middle — it holds no viewer logic of its own.

The webview's HTML is static; everything it cannot know (resource URIs, your settings) arrives in an init message when the panel opens. `ocp-viewer-core` and `three-cad-viewer` are npm dependencies of the extension, shipped inside the `.vsix`.

## Python to CAD Viewer

The Python side connects over a local WebSocket (`ws://127.0.0.1:<port>`) — one connection per message, so a viewer restarted between two shows just works. The extension relays models and configs into the webview via `postMessage`, and answers the two questions a show asks: `config` from the VS Code settings, `status` from the last state the webview pushed (the viewer reports every change as it happens, so the answer is a cache, not a round trip into the browser). The port is what [addresses](addressing.md) one of several viewers.

## Where the backend runs and how measurements happen

The [measurement backend](../../concepts/backend.md) runs as its **own Python process**, started by the extension (`python -m ocp_vscode --backend --port N`). It registers with the extension's WebSocket server as a listener; after each show it receives the model's id-to-shape mapping. When you pick shapes with an analysis tool, the webview's notification travels to the extension, on to the backend process, which computes the exact answer from the BRep and sends it back on a second connection — extension → webview → the panel on your screen.
