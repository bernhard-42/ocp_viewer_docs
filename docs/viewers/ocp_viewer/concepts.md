# Concepts

How the shared pieces are wired in the standalone viewer. The general mechanisms are in the [Concepts](../../concepts/architecture.md) chapter; this page is OCP Viewer's shape of them.

## The integration chain

```text
your Python process ──ws──▶ python -m ocp_viewer ──ws──▶ browser tab
     show()                  (Flask + websockets,          ocp-viewer-core JS page
                              relay + state + backend)       └─▶ three-cad-viewer
```

Three processes: your Python process runs the core's show pipeline; the **server** (`python -m ocp_viewer`) relays messages and keeps the viewer's state; the **browser tab** runs the core's JavaScript page embedding three-cad-viewer, served by that same server together with its static copies of the JS packages.

## Python to CAD Viewer

Your script connects to the server over a local WebSocket, the browser page holds a second WebSocket to the same server, and models are relayed verbatim from one to the other. The server answers the two questions a show asks: `config` from `~/.ocpvscode_standalone` plus the CLI flags, `status` from the state the page continuously pushes up. If the server goes away, the page keeps trying to reconnect (`--max_reconnect_attempts`). The port is what [addresses](addressing.md) one of several servers.

## Where the backend runs and how measurements happen

The [measurement backend](../../concepts/backend.md) lives **inside the server process**. After each show it receives the model's id-to-shape mapping; when you pick shapes with an analysis tool, the browser's notification arrives on its WebSocket, the server hands it to the backend, and the exact answer travels back on the same socket into the page. One socket, both directions — the simplest of the four arrangements.
