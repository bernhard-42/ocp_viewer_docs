# Python to CAD Viewer communication

Every viewer answers the same question differently: how does a model get from the Python process running your script into the CAD Viewer on your screen? The *what* is shared; only the *how* is per viewer.

## The contract

The core defines a small transport contract every viewer implements — the four things Python ever initiates:

| message | carries |
| ------- | ------- |
| data | a model to draw |
| config | a configuration block to apply |
| command | a request with a reply: `status`, `config`, a screenshot, the animation clock |
| backend | the id-to-shape mapping of the model just drawn, for the [measurement backend](backend.md) |

And three that come back: updates (what changed in the viewer), listen (a receiver registering to be sent things), and the backend's measurement responses. Viewers that speak over a socket spell these as one-letter prefixes (`D:`, `C:`, `U:`, `L:`, `B:`, `R:`, `S:`); only the socket layer knows that.

**The sender translates to the receiver's paradigm.** Python speaks snake_case and enums; the renderer speaks camelCase and plain JSON. The conversion happens once, at the wire, on the way out — nothing on the JavaScript side renames a key or accepts two spellings, so a mistake surfaces as an unknown key instead of being quietly tolerated.

## The four transports

| Viewer | Transport | Who is in the middle |
| ------ | --------- | -------------------- |
| VS Code CAD Viewer | WebSocket to the extension, which relays into the webview | the extension host is the server |
| OCP Viewer | WebSocket to the Flask server, relayed verbatim to the browser page | the server |
| Jupyter CadQuery | ipywidgets traitlets over the Jupyter comm channel | the Jupyter server's widget machinery |
| build123d Studio | IPC between kernel, sidecar and the app's webview | the sidecar |

Jupyter CadQuery is the interesting outlier: a traitlet has the same name in Python and JavaScript, so its transport skips the camelCase translation — the rule is unchanged, the sender still translates to the receiver's paradigm, and there the two paradigms happen to be one.

Each viewer's own Concepts page draws its chain in full.

## One caveat worth knowing

Viewer-to-Python change notification is **asynchronous**. A tree toggle made while a cell is running may not have reached Python by the time that cell calls `show()` — so "every show first reads the viewer state and reapplies it" is a best effort, not a promise. A stale status is merged, never depended on.
