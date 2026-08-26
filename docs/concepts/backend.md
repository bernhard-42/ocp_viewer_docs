# The measurement backend

The [analysis tools](../measure_select.md) show two kinds of numbers: an instant mesh-based hover readout, and CAD-exact panel values. The exact ones come from the **measurement backend** — a piece of Python that holds the *BRep geometry* of the last shown model, while the CAD Viewer only ever holds its tessellation.

## The mechanism

After every `show`, the id-to-shape mapping of what was just drawn is handed to the backend. When you then pick shapes with a tool, the same five steps run in every viewer:

1. the CAD Viewer notifies which shape ids are selected and which tool is active
2. the notification travels to the Python that holds the model — *this is where viewers differ*
3. that Python hands the change set to the backend
4. the backend computes exact geometry from the BRep — a distance with its points, a face's area, a solid's volume — and returns it, or nothing when the selection doesn't match what the tool needs
5. the answer is sent back toward the CAD Viewer on the channel the request came in on — *this is where viewers differ again*

The backend itself has no transport: it answers by returning. Whoever delivered the notification already holds the channel the answer goes back on.

## Where it runs, per viewer

| Viewer | Where the backend lives |
| ------ | ----------------------- |
| VS Code CAD Viewer | its own process, started by the extension, connected over a listener websocket |
| OCP Viewer | inside the server process |
| Jupyter CadQuery | inside a Jupyter *server* extension, reached over HTTP |
| build123d Studio | its own measurement process beside the kernel |

Each viewer's Concepts page has the full picture. When no backend is reachable, the tools fall back to mesh-based values and the panels title themselves "(mesh based)" — approximate, but honest about it.
