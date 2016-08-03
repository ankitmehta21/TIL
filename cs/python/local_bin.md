I had some trouble installing [bugzillatools](https://pypi.python.org/pypi/bugzillatools/0.5.3.1) using pip today.
The standard pip install command worked fine, but the binary wasn't in my path for some reason.
Looks like pip was installing to `~/.local/bin/`.
Adding this to my .bashrc was simple:

```bash
# Add python local path if it exists
if [ -d "$HOME/.local/bin" ]; then
    export PATH=$PATH:$HOME/.local/bin
fi
```

And now I'm off.
