# Jupyter Console

When running code cell by cell with the Jupyter extension, the sidebar's **Open Jupyter Console** button attaches a real `jupyter console` to the same kernel your cells run on — so you can inspect and poke at the objects your cells created, from a terminal, without disturbing the notebook state.

The plumbing is automatic: when the Python side connects to the viewer from inside a Jupyter kernel, it writes the kernel's connection file path into the `~/.ocpvscode` [registry](addressing.md#the-registry-ocpvscode) next to the viewer's port — that entry is what the console attaches to.

- `set_connection_file()`

    Refresh that registry entry by hand. Called automatically on first use; only invoke it directly if the kernel was restarted and the console can no longer attach.

The `jupyter_console` package itself is one of the libraries the [Library Manager](installation.md#the-library-manager) can install.
