# Visual debugging

With visual debugging on, every debugger step checks all variables in `locals()` for CAD objects and shows them in the viewer under their variable names — stepping through a model is watching it being built.

## Using it

- Set breakpoints and step through the code as usual with VS Code's Python debugger.
- Check that `OCP: <port>·DEBUG` is visible in the status bar; clicking it toggles between `OCP: <port>·DEBUG` (visual debugging enabled) and `OCP: <port>` (disabled). Whether it starts enabled is the `OcpCadViewer.advanced.watchByDefault` setting.

    ![OCP:on](../../assets/ocp-on.png)

- Planes, locations and axes are shown too — name your build123d contexts (including `Location` contexts) so they appear with meaningful labels.
- The viewer remembers camera position and which variables were unselected in the tree across steps (e.g. to hide temp variables that are out of scope).

What runs on each step is the `OcpCadViewer.advanced.watchCommands` setting — it must call `show_all(locals(), ...)`, and customizing it customizes what a debug step shows.

## Python pdb

The same visual stepping works in plain `pdb`, by hooking `show_all` into every stop:

```python
def show_all_locals(self, stop, line): return stop if stop else self.default("from ocp_vscode import show_all, get_port; show_all(locals(), port=get_port(), _visual_debug=True)")
import pdb; pdb.Pdb.postcmd = show_all_locals
```
