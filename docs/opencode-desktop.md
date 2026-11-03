# OpenCode desktop: PATH and the Obsidian CA

The desktop app starts a background service (`opencode-cli serve --service`) with
the environment it was launched with. That environment is not your shell: on a GUI
launch it is typically `/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin`. Two
things break as a result:

- `npx`-based MCPs (`figma`, `chrome-devtools`) fail with `NotFound: npx`.
- The Obsidian MCP fails with `DEPTH_ZERO_SELF_SIGNED_CERT`.

A config reload re-spawns MCP children but does not change the service environment.
Fully restart the app after changing this.

## Launcher wrapper

Create `~/.local/bin/opencode-desktop`:

```sh
#!/bin/sh
export PATH="$HOME/.local/share/fnm/aliases/default/bin:$HOME/.opencode/bin:$HOME/.local/bin:$PATH"

VAULT="${OBSIDIAN_VAULT:-$HOME/Documents/Obsidian/Engineering}"
DATA="$VAULT/.obsidian/plugins/obsidian-local-rest-api/data.json"
CERT="$HOME/.config/opencode/certs/obsidian-local-rest-api.pem"

if [ -f "$DATA" ] && command -v jq >/dev/null 2>&1; then
  mkdir -p "$(dirname "$CERT")"
  if jq -r '.crypto.cert' "$DATA" >"$CERT" 2>/dev/null && [ -s "$CERT" ]; then
    :
  else
    rm -f "$CERT"
  fi
fi
[ -s "$CERT" ] && export NODE_EXTRA_CA_CERTS="$CERT"

exec /opt/OpenCode/ai.opencode "$@"
```

Then `chmod 755 ~/.local/bin/opencode-desktop`. Adjust `/opt/OpenCode/ai.opencode`
to the app executable on the platform; the Linux desktop package uses that path.

## Point the desktop entry at it

The app ships `ai.opencode.desktop.desktop` and `opencode-desktop.desktop`. Create
user-level overrides so the menu uses the wrapper:

```sh
mkdir -p ~/.local/share/applications
for f in ai.opencode.desktop.desktop opencode-desktop.desktop; do
  cp "/usr/share/applications/$f" ~/.local/share/applications/"$f"
  sed -i "s#^Exec=/opt/OpenCode/ai.opencode#Exec=$HOME/.local/bin/opencode-desktop#" \
    ~/.local/share/applications/"$f"
done
update-desktop-database ~/.local/share/applications
```

Quit OpenCode fully and relaunch it from the menu. A reload is not enough.

## Obsidian certificate

The plugin's `data.json` carries the self-signed certificate as `crypto.cert`. The
wrapper extracts it on every launch. To do it manually:

```sh
mkdir -p ~/.config/opencode/certs
jq -r '.crypto.cert' \
  "$HOME/Documents/Obsidian/Engineering/.obsidian/plugins/obsidian-local-rest-api/data.json" \
  > ~/.config/opencode/certs/obsidian-local-rest-api.pem
chmod 644 ~/.config/opencode/certs/obsidian-local-rest-api.pem
```

The certificate is public; the token at `~/.config/secrets/obsidian-token` is not.
If the plugin regenerates the certificate, re-extract it (the wrapper does this
automatically).

## Verify

```sh
opencode mcp list
```

All configured servers should report `connected`. Then make a real tool call, for
example `list_pages` for chrome-devtools or any Obsidian tool; a connected server
is not proof that the browser or token works.

To inspect the service's actual environment:

```sh
pid=$(pgrep -f 'opencode-cli serve --service' | head -1)
tr '\0' '\n' < /proc/"$pid"/environ | grep -E '^PATH=|^NODE_EXTRA_CA_CERTS='
```

You want the fnm, `~/.opencode/bin`, and `~/.local/bin` directories on `PATH`, and
`NODE_EXTRA_CA_CERTS` pointing at the extracted certificate.

## Terminal CLI

If you launch `opencode` from a terminal instead, add the same path directories to
`~/.zshenv` and export `NODE_EXTRA_CA_CERTS` there. The launcher wrapper is only for
the GUI app.
