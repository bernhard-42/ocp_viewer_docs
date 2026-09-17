# Troubleshooting

Three places say what happened: the toolbar's health chip, the **Backend** tab, and the log files.

## The health chip

The toolbar shows a status chip only when there is something to say. The model channel that carries `show()`, the measurement process, the kernel, the language server and the console each have a lifecycle of their own, and any of them can fail while the window looks well — `show()` draws nothing, a measurement answers nothing, completion goes silent.

The chip shows the worst state of any subsystem, `degraded` (working, with something to know) or `failed` (not working). Its tooltip lists every subsystem with what it last reported. Click it to open the **Backend** tab, where the reason is.

## The Backend tab

The bottom pane's third tab is what the measurement process reports: which shape was clicked, what it indexed, why a measurement could not be taken. Read-only; the same lines go to `backend.log`.

A subsystem that newly fails brings this tab forward once.

## The recovery banner

Two failures are reported in the window, each with the button that fixes it:

- _"The Python backend disconnected."_ → **Restart backend**. The sidecar process is gone, and everything Python-side with it.
- _"The Python kernel stopped. Variables and imports are gone."_ → **Restart kernel**. The sidecar is fine; the kernel died.

Neither restarts on its own.

## The log files

Three files in the [data directory](first_run.md#where-it-all-lives), outside the environment so a rebuild of it keeps them:

| File                   | What is in it                                                                                                                                                                                                                                                                  |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `build123d-studio.log` | Everything the application reports about itself: the machine it runs on, the kernel's lifecycle, what the sidecar did, uv's output while the environment is built, every command's exit code, every warning the frontend raises. The first thing to attach to a report.       |
| `console.log`          | The embedded browser's own output — a renderer complaining about a malformed model, a failed request, an uncaught error with its stack.                                                                                                                                     |
| `backend.log`          | What the **Backend** tab shows.                                                                                                                                                                                                                                                |

**Each is rotated at every start**: this session's file is the plain name, the previous session's is `.1`, nothing older is kept. After a crash the file you want is usually `build123d-studio.log.1`.

How much of the browser's output is captured is **Settings → Application → Debug console**: `Nothing`, `Errors` (the default), `Errors and warnings`, or `Everything`. Turn it up to reproduce something, then back down — a chatty library fills the file.

!!! note "Why there is a console log file"

    On macOS there is no developer console to open: Safari lists an embedded web view only when the application marks it inspectable, which this toolkit does not do. On Windows and Linux the embedded browser's own **Inspect** entry may be available by right-clicking.

## About, and what a report should carry

**Help → About** gathers it in one place, with a **Copy** button at the top that yields the whole dialog as text:

- the application version, the platform, and the versions of Monaco, three-cad-viewer, ocp-viewer-core, xterm.js and uv
- the Python environment's location, its interpreter, and this instance's Jupyter connection file
- the key package versions
- every log file that exists, including the rotated one, and the snippets file

Every path has a **Copy** button beside it; every file has an **Open** button that opens it in the editor.

## Common situations

| Situation                                                    | What to do                                                                                                                                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| The first start is blocked by the OS                         | See [First Run](first_run.md#getting-past-the-os-gatekeepers) for the per-platform way through                                                                                 |
| The first start takes a long time                            | Expected: it downloads a pinned uv and a pinned CPython and builds an environment of several hundred megabytes. The splash says which step it is on                            |
| The first start fails on Windows with a French, German… "path not found" line | The command prompt's AutoRun registry entry names a script that no longer exists (an uninstalled Anaconda leaves one). The splash says so; delete the value under `HKCU\Software\Microsoft\Command Processor` |
| The environment is broken beyond repair                      | Delete the environment folder named in About; the next start rebuilds it                                                                                                       |
| A line in **Additional packages** breaks the environment     | The application still starts: that section is dropped for the launch and the text stays in Settings to be corrected. See [Packages](packages.md#additional-packages)          |
| Completion or squiggles are stale after installing something | The environment actions restart the language server. If it did not come back, the splash said so; re-open Settings and press **Update packages** again                        |
| `show()` draws nothing                                       | Check the health chip: the model channel and WebGL are among the subsystems it reports on                                                                                      |
| Measurements answer nothing                                  | The **Backend** tab is that process talking                                                                                                                                    |
| `studio` on the command line cannot find the application     | Start the application once first; the launcher reads a location the application records on every start                                                                        |
| Unsaved work after a crash                                   | It is offered back at the next start. The copies are ordinary source files in the data directory                                                                               |
| The machine refuses to run anything from the environment     | Set `BUILD123D_STUDIO_ENV_ROOT` to a directory the policy allows — see [Moving the environment](first_run.md#moving-the-environment)                                           |
