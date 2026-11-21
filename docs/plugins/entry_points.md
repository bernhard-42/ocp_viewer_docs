# Registration and entry points

Implementing the protocols is not enough: the viewer has to find the plugin, in two processes, neither of which imports it. This page is about that second half, and it decides whether a finished plugin does anything at all. It takes two things: **registering** each half of the plugin when its module is imported, and declaring two **entry points** in `pyproject.toml`, which make the viewer import those modules.

## Why a plugin needs entry points

ocp-tessellate and ocp-viewer-core do not know any plugin by name, since the plugin depends on them and not the other way round. And no one else imports the plugin either:

- **The user's script** imports the kernel and the host, not the plugin: `from smlibpy import Box` and `from ocp_vscode import show`. A debug stop sees whatever the script holds.
- **The measurement backend** runs in [another process](backend.md#where-the-backend-runs) in every host, and imports no code of the user's.

An entry point solves both. It is a standard mechanism of Python packaging: a package declares in its metadata a line of the form `name = value` in a named group, and any other package can read that group from the installed packages' metadata, without importing them and without knowing their names.

## Register when imported

A plugin has two halves, and each process needs one of them: the user's process draws and serializes, the backend's process deserializes and measures. So there are two modules, and importing each registers its half. In ocp-viewer-smlibpy they are the last lines of `show.py` and `backend.py`:

```python
# ocp_viewer_smlibpy/show.py
register_show(KERNEL, SmlibTessellation(), SmlibShow(), api=1)

# ocp_viewer_smlibpy/backend.py
register_plugin(KERNEL, SmlibBackend(), api=1)
```

Keep the two apart: neither module imports the other, and the package's `__init__.py` imports neither, since Python runs it before any module of the package. What both need, the tag and the format version of the bytes, goes into a third module, `_wire.py` in ocp-viewer-smlibpy.

`KERNEL` is the plugin's **kernel tag**, here `"smlib"`, under which all three protocols are registered. It is the name the kernel goes by in the backend's model and in the log. An empty tag and `"OCCT"`, which names OCCT's own shapes, are refused with a `ValueError`.

`api` is the version of the plugin interface the plugin was written against, as a number of its own. It is a required keyword, and `1` is the only version so far. A later version of the interface passes new arguments only to plugins that state it, so a plugin keeps working as it is. Today it refuses a plugin that needs a newer ocp-tessellate or ocp-viewer-core, with a `ValueError`: `Kernel plugin 'smlib' was written for plugin interface 2, this ocp-tessellate serves interface 1 to 1`. Anything but an `int` is a `TypeError`.

Both functions raise a `TypeError` for an object that does not implement its protocol. Each prints one line, so each process says what it registered:

```
Kernel plugin 'smlib' registered for drawing (ocp_viewer_smlibpy.show)
```

```
Kernel plugin 'smlib' registered for measuring (ocp_viewer_smlibpy.backend)
```

A second registration of the same tag replaces the first and says `re-registered`. Importing these modules should do nothing beyond registering, since every process that shows anything imports every installed plugin's show module.

## Declare the entry points

```toml
[project.entry-points."ocp_viewer.kernels.show"]
smlib = "ocp_viewer_smlibpy.show"

[project.entry-points."ocp_viewer.kernels.backend"]
smlib = "ocp_viewer_smlibpy.backend"
```

In both groups the key is the kernel tag, and the value is the module whose import registers.

| | group `ocp_viewer.kernels.show` | group `ocp_viewer.kernels.backend` |
|---|---|---|
| read by | ocp-tessellate | ocp-viewer-core |
| in which process | the user's | the backend's |
| what is imported, and when | all plugins, at the first show | the plugin a model names, at the first model that carries its tag |

## What happens at runtime

In the user's process:

1. The first time a show meets an object, of any kernel and OCCT too, ocp-tessellate imports all plugins of the group `ocp_viewer.kernels.show`, once per process. Each import registers its show half.
2. It asks each registered plugin's [`is_object`](show.md#is_object) about the object. The plugin that answers `True` owns it, and the object goes through `convert` and `tessellate`. A raw OCCT shape is never a plugin's.
3. Before anything is sent, each shape of the backend's model becomes bytes through the plugin's [`serialize`](show.md#serialize), tagged with `smlib`. If that fails, the show raises and the viewer and the backend stay on the scene they had.

In the backend's process, when that model arrives:

1. The backend reads the tag `smlib` from a leaf of the model, and finds nothing registered under it.
2. It looks up that key in the group `ocp_viewer.kernels.backend`, and imports the value, `ocp_viewer_smlibpy.backend`. Nothing of the plugin's drawing is imported.
3. The import registers the protocol, and the backend rebuilds the shape from the bytes with the plugin's [`deserialize`, `moved` and `parts`](backend.md#backendprotocol).

## Installing

Entry points are read from the **installed** package's metadata (`entry_points.txt` in its `.dist-info` folder), not from `pyproject.toml`. So a plugin folder that is merely on `sys.path`, without that metadata, has no entry points, and:

- the plugin has to be installed, `pip install -e .` counts;
- after adding or changing an entry point the package has to be installed again, also for an editable install, since that is when the metadata is written;
- the plugin has to be installed in both environments, the one of the user's script and [the one the backend runs in](backend.md#where-the-backend-runs), which need not be the same;
- ocp-tessellate reads its group once per process, so a process that has already shown something, such as a Jupyter kernel, needs a restart after the install.

To see what an environment declares:

```bash
python -c "from importlib.metadata import entry_points as e; print(e(group='ocp_viewer.kernels.show')); print(e(group='ocp_viewer.kernels.backend'))"
```

```
EntryPoints((EntryPoint(name='smlib', value='ocp_viewer_smlibpy.show', group='ocp_viewer.kernels.show'),))
EntryPoints((EntryPoint(name='smlib', value='ocp_viewer_smlibpy.backend', group='ocp_viewer.kernels.backend'),))
```

## When it does not work

| what you see | what is wrong |
|---|---|
| `show` prints `show: skipped smlibpy.shapes.Part (no kernel plugin installed?)` and draws the other objects, passed directly or inside a list, tuple or dict; `show_all` leaves the object out without a line | the entry point of the group `ocp_viewer.kernels.show` is missing, or the plugin is not installed in the script's environment |
| The warning `Kernel plugin 'ocp_viewer_smlibpy.show' ('smlib') failed to load: ...`, then the same `show: skipped` line | the entry point's value names a module that is missing, or its import raises. The warning gives the reason, and everything else is still shown |
| The warning `Kernel plugin 'ocp_viewer_smlibpy.show' was imported, but did not register its kernel tag 'smlib' with register_show` | the key of the entry point is not the tag the plugin registers, or the module does not call `register_show` when imported |
| The warning `is_object of kernel plugin 'smlib' raised: ...`, once per plugin | the plugin's `is_object` raises. For that question the object counts as not the kernel's |
| `ValueError: Kernel plugin 'smlib' was written for plugin interface 2, ...` | the plugin needs a newer ocp-tessellate or ocp-viewer-core |
| `show` raises what the plugin's `serialize` raised, and nothing is drawn | the plugin cannot write this shape. The viewer and the backend stay on the scene they had |
| The objects are drawn, and the backend logs `smlib shape '/Group/plate' not loaded: LookupError("no measurement backend plugin for kernel 'smlib': ... Registered: [...]")` | the plugin is not installed in the backend's environment, or the key of the entry point in the group `ocp_viewer.kernels.backend` is not the tag. Everything else of the scene is still measured; a click on this shape logs `unknown identifier received` |
| The objects are drawn, and the backend logs `smlib shape '/Group/plate' not loaded: ModuleNotFoundError(...)`, or another error | the entry point's value names a module that is missing there, its import raises, or the plugin's `deserialize`, `moved` or `parts` raises for this shape, for example on a format version it cannot read |

An explicit `import ocp_viewer_smlibpy.show` in a script registers the drawing half too, so a missing entry point goes unnoticed there. The backend's process has no such import, and always needs its entry point.

[Toy plugins and testing](examples.md#what-to-test) names the tests that load a plugin through its entry points in a fresh process.
