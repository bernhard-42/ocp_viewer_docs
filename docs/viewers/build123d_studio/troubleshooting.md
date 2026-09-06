# Troubleshooting

Three places say what happened, and between them they cover most of it: the **Backend** tab, the toolbar's health chip, and the log files.

## The health chip

The toolbar carries a status chip only when there is something to say. Everything below the window has a lifecycle of its own — the model channel that carries `show()`, the measurement process, the kernel, the language server, the console — and any of them can die while the window still looks entirely well. That is the expensive failure: `show()` draws nothing, a measurement answers nothing, completion goes silent, and every symptom afterwards has the wrong shape.

The chip shows the worst state anything is in, `degraded` (working, with something worth knowing) or `failed` (not working). Its tooltip lists every subsystem with what it last said. Click it to open the **Backend** tab, where the reason is.

## The Backend tab

The bottom pane's third tab is the measurement process's own account, as it says it: which shape id was clicked, what it indexed, why a measurement could not be taken. It runs in a process of its own, its troubles are frequent, and a log file you have to go looking for is the wrong first place for them. Read-only; the same lines also go to `backend.log`.

A subsystem that newly fails brings this tab forward on its own — once per subsystem, not every time it speaks again.

## The recovery banner

Two deaths are reported in the window rather than only in the log, each with the button that fixes it:

- _"The Python backend disconnected."_ → **Restart backend**. The sidecar process is gone and everything Python-side went with it.
- _"The Python kernel stopped. Variables and imports are gone."_ → **Restart kernel**. The sidecar is fine; only the kernel died.

Neither restarts itself. A backend that restarts on its own turns one crash into a loop of them with the log filling up behind it.

## The log files

Three files, three subjects, all in the [data directory](first_run.md#where-it-all-lives) — deliberately not inside the environment, so they survive a rebuild of it:

| File                   | What is in it                                                                                                                                                                                                                                                                  |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `build123d-studio.log` | Everything the application reports about itself: the kernel's lifecycle, what the sidecar did, uv's own output while the environment is built, every warning the frontend raises. This is the first thing to attach to a report.                                               |
| `console.log`          | The browser's own output — what the embedded browser and the libraries inside it print: a renderer complaining about a malformed model, a failed request, an uncaught error with its stack. Separate on purpose, so turning it up does not bury the application's own account. |
| `backend.log`          | What the **Backend** tab shows.                                                                                                                                                                                                                                                |

**Each is rotated at every start, not truncated**: this session's file is the plain name, the previous session's is `.1`, and nothing older is kept. So after a crash the file you want is usually `build123d-studio.log.1` — the one from before the restart that made the problem obvious.

How much of the browser's output is captured is **Settings → Application → Debug console**: `Nothing`, `Errors` (the default and the useful setting), `Errors and warnings`, or `Everything`. Turn it up to reproduce something specific, and turn it back down — a chatty library fills the file and pushes out the line you need.

!!! note "Why there is a console log file at all"

    On macOS there is no developer console to open: Safari lists an embedded web view only when the application marks it inspectable, which macOS has required since 13.3 and this toolkit does not do. On Windows and Linux the embedded browser's own **Inspect** entry may be available by right-clicking, and the window is configured to allow it.

## About, and what a report should carry

**Help → About** gathers it all in one place, with a **Copy** button that yields the whole thing as plain text:

- the application version, the platform, and the versions of Monaco, three-cad-viewer, ocp-viewer-core, xterm.js and uv
- the Python environment's location, its interpreter, and this instance's Jupyter connection file
- the key package versions, under the names people say out loud
- every log file that exists, including the rotated one, and the snippets file

## Common situations

| Situation                                                    | What to do                                                                                                                                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| The first start is blocked by the OS                         | Signing and notarisation are not in place yet — see [First Run](first_run.md#getting-past-the-os-gatekeepers) for the per-platform way through                                 |
| The first start takes a long time                            | Expected: it downloads a pinned uv and a pinned CPython and builds an environment of several hundred megabytes. The splash says which step it is on                            |
| The environment is broken beyond repair                      | Delete the environment folder named in About; the next start rebuilds it from scratch                                                                                          |
| A line in **Additional packages** breaks the environment     | The application still starts — the failed section is dropped for that launch and the text is left in Settings to be corrected. See [Packages](packages.md#additional-packages) |
| Completion or squiggles are stale after installing something | The three environment actions restart the language server themselves. If it did not come back, the splash said so; re-open Settings and press **Update packages** again       |
| `show()` draws nothing                                       | Check the health chip: the model channel is one of the subsystems it reports on                                                                                                |
| Measurements answer nothing                                  | The **Backend** tab is that process talking                                                                                                                                    |
| `studio` on the command line cannot find the application     | Start the application once first — the launcher finds it by reading a location the application records on every start                                                          |
| Unsaved work after a crash                                   | It is offered back at the next start. The copies are ordinary source files in the data directory, openable in any editor                                                       |
| The machine refuses to run anything from the environment     | Set `BUILD123D_STUDIO_ENV_ROOT` to a directory the policy allows — see [Moving the environment](first_run.md#moving-the-environment)                                           |
