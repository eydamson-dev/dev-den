# Dev Den inventory

Every item the skill knows about. Each item has:

- **Applies to:** `desktop`, `laptop`, or `both`. A GPU-only item is skipped on a
  machine without a GPU.
- **Mode:** `AI` (the AI runs it), `CMD` (I run the exact command the AI prints), or
  `GUI` (I follow written steps and say `done`).
- **Needs:** items that must be `working` first.
- **Reference:** the version this was verified on, from the desktop. Used as the
  safe reference. Not a pin. Use the latest stable version when a reference is not
  needed for the setup to work.

Commands use placeholders for anything machine-specific. Fill them from the
private machine note, never from this file.

---

## 1. Base system

| Item | Applies to | Mode | Check | Install | Reference |
| --- | --- | --- | --- | --- | --- |
| Fedora | both | GUI | `grep -q '^VERSION_ID=44' /etc/os-release` | installed by OS | Fedora 44 |
| git | both | AI | `git --version` | `sudo dnf install -y git` | — |
| git-lfs | both | AI | `git lfs version` | `sudo dnf install -y git-lfs` | — |
| curl, wget | both | AI | `curl --version`, `wget --version` | `sudo dnf install -y curl wget` | — |
| openssh-clients | both | AI | `ssh -V` | `sudo dnf install -y openssh-clients` | — |
| jq | both | AI | `jq --version` | `sudo dnf install -y jq` | — |
| python3, pip | both | AI | `python3 --version && python3 -m pip --version` | `sudo dnf install -y python3 python3-pip` | Python 3.14 (uv-managed venvs) |
| build tools | both | AI | `gcc --version` | `sudo dnf group install -y development-tools` | — |

## 2. Secrets directory

| Item | Applies to | Mode | Check | Install | Notes |
| --- | --- | --- | --- | --- | --- |
| secrets directory | both | AI | `stat -c '%a' ~/.config/secrets` prints `700` | `mkdir -p ~/.config/secrets && chmod 700 ~/.config/secrets` | Never print values |
| Figma token | both | CMD | `test -s ~/.config/secrets/figma-token` | `printf '%s' 'TOKEN' > ~/.config/secrets/figma-token && chmod 600 ~/.config/secrets/figma-token` | Raw file, no trailing newline |
| Obsidian REST token | both | GUI | `test -s ~/.config/secrets/obsidian-token` | Copy from the plugin settings, then `printf '%s' 'KEY' > …` | Generated per install. Never copy from another machine |
| Proxmox token | both | CMD | `test -s ~/.config/secrets/proxmox-token` and mode `600` | See section 13 | `TOKEN_NAME` + `SECRET` lines, not a raw value. The MCP env maps them to `PROXMOX_TOKEN_NAME`/`PROXMOX_TOKEN_VALUE` |
| homelab SSH key | both | CMD | `test -f ~/.ssh/<homelab-key>` | `ssh-keygen -t ed25519 -f ~/.ssh/<homelab-key> -C "<machine-name>"` | One key per machine. Add the public key to the homelab yourself |

Each secret is a separate raw file. There is no shared `~/.config/secrets/env`
file; do not create one.

## 3. Git and GitHub

| Item | Applies to | Mode | Check | Install | Needs |
| --- | --- | --- | --- | --- | --- |
| Git identity | both | CMD | `git config --global user.name && git config --global user.email` | `git config --global user.name "<name>"` and `git config --global user.email "<email>"` | git |
| GitHub CLI | both | AI | `gh --version` | `sudo dnf install -y gh` | — |
| GitHub auth | both | CMD | `gh auth status` | `gh auth login --hostname github.com --git-protocol https --web` | gh |
| SSH agent | both | CMD | `ssh-add -l` | `eval "$(ssh-agent -s)" && ssh-add ~/.ssh/<homelab-key>` | ssh key |

## 4. Containers

| Item | Applies to | Mode | Check | Install | Reference |
| --- | --- | --- | --- | --- | --- |
| Docker CE | both | AI | `docker --version` | `sudo dnf install -y dnf-plugins-core`, then `sudo dnf config-manager addrepo --from-repofile=https://download.docker.com/linux/fedora/docker-ce.repo`, then `sudo dnf install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin` | Docker 29.x |
| Docker daemon | both | AI | `systemctl is-active docker` prints `active` | `sudo systemctl enable --now docker` | — |
| Docker access | both | CMD | `docker info >/dev/null 2>&1` | `sudo usermod -aG docker "$USER"`, then log out and back in | Group change needs a new login. A new shell is not enough |
| Docker Compose | both | AI | `docker compose version` | included with docker-compose-plugin | — |
| Chromium browser image | both | AI | `docker image inspect jlesage/chromium:latest` | `docker pull jlesage/chromium:latest` | Used by chatgpt-bridge |

## 5. Node and Python tooling

| Item | Applies to | Mode | Check | Install | Reference |
| --- | --- | --- | --- | --- | --- |
| fnm | both | AI | `fnm --version` | `curl -fsSL https://fnm.vercel.app/install \| bash` | — |
| Node.js LTS | both | AI | `node --version` (after `eval "$(fnm env)"`) | `fnm install --lts && fnm default lts-latest` | Node via fnm only, not the system package |
| pnpm | both | AI | `pnpm --version` | `npm install -g pnpm` | pnpm 12.x |
| uv | both | AI | `uv --version` | `curl -LsSf https://astral.sh/uv/install.sh \| sh` | — |
| uvx | both | AI | `uvx --version` | installed with uv | Needed by the Proxmox MCP and Blender MCP |

## 6. Developer CLI tools

| Item | Applies to | Mode | Check | Install | Reference |
| --- | --- | --- | --- | --- | --- |
| ripgrep | both | AI | `rg --version` | `sudo dnf install -y ripgrep` | — |
| fd | both | AI | `fd --version` | `sudo dnf install -y fd-find` (the binary is `fd`) | — |
| fzf | both | AI | `fzf --version` | `sudo dnf install -y fzf` | — |
| bat | both | AI | `bat --version` | `sudo dnf install -y bat` | — |
| lazygit | both | AI | `lazygit --version` | `sudo dnf copr enable dejan/lazygit`, then `sudo dnf install -y lazygit` (not in the Fedora repos) | — |
| tmux | both | AI | `tmux -V` | `sudo dnf install -y tmux` | — |
| stylua | both | AI | `stylua --version` | `cargo install stylua` or the upstream release | Neovim formatter |
| pre-commit | both | AI | `pre-commit --version` | `uv tool install pre-commit` | — |
| Coolify CLI | both | CMD | `coolify version` | Install from the Coolify CLI release page, put it in `/usr/local/bin` | 1.11.0 verified on the laptop; context in the private note |
| Cloudflare `cf` CLI (beta) | both | AI | `cf --version` | `npm install -g cf` | Beta. Install only. Do not log in as part of setup |
| codex CLI | desktop | CMD | `codex --version` | Install from its release page | Optional |

Coolify context: `coolify context add <name> <url> <token> --default`, then
`coolify context verify`. Keep the token in `~/.config/secrets/coolify-token` as
`URL=`/`TOKEN=`; do not pass it in argv.

## 7. Shell

| Item | Applies to | Mode | Check | Install | Notes |
| --- | --- | --- | --- | --- | --- |
| zsh | both | AI | `zsh --version` | `sudo dnf install -y zsh` | — |
| default shell is zsh | both | CMD | `getent passwd "$USER" \| grep -q zsh` | `chsh -s "$(command -v zsh)"` | Needs a new login |
| oh-my-zsh | both | AI | `test -d ~/.oh-my-zsh` | official installer, unattended | — |
| zsh-autosuggestions | both | AI | `test -d ~/.oh-my-zsh/custom/plugins/zsh-autosuggestions` | `git clone https://github.com/zsh-users/zsh-autosuggestions ~/.oh-my-zsh/custom/plugins/zsh-autosuggestions` | — |
| fzf key bindings | both | AI | `test -f /usr/share/fzf/shell/key-bindings.zsh` | `sudo dnf install -y fzf` (ships the file); source it from `~/.zshrc` | Fedora path. Verify the path after install |

Optional, not on the desktop: starship, zoxide, atuin, eza, direnv. Offer them, do
not install them without a yes.

## 8. Editor and terminal config

| Item | Applies to | Mode | Check | Install | Notes |
| --- | --- | --- | --- | --- | --- |
| Neovim | both | AI | `nvim --version` | `sudo dnf install -y neovim` | — |
| Neovim config | both | AI | `test -f ~/.config/nvim/init.lua` | copy from the dotfiles repository | Plugins install on first launch |
| tmux config + TPM | both | AI | `test -d ~/.config/tmux/plugins/tpm` | `git clone https://github.com/tmux-plugins/tpm ~/.config/tmux/plugins/tpm` | Then press prefix + I in tmux |

## 9. OpenCode

| Item | Applies to | Mode | Check | Install | Reference |
| --- | --- | --- | --- | --- | --- |
| OpenCode CLI | both | AI | `opencode --version` | `curl -fsSL https://opencode.ai/install \| bash` | 2.2.5 (verified); 2.0.29 was the earlier reference |
| global config | both | AI | `jq empty ~/.config/opencode/opencode.json`, then `./setup.sh verify --only opencode` reports no drift | Render from `~/projects/dotfiles`: `./setup.sh configure --only opencode`; machine choices live in `~/.config/dev-den/machine.json`. Never hand-merge | New servers are gated by `config/features.json` + the profile |
| global AGENTS.md | both | AI | `test -s ~/.config/opencode/AGENTS.md` | `./setup.sh configure --only opencode` from `~/projects/dotfiles` | Machine-specific text lives in `~/.config/dev-den/local/AGENTS.md` |
| `tools` glob for LAN browser | desktop + laptop | AI | `jq -e '.tools["obscura-dev_*"] == false' ~/.config/opencode/opencode.json` | Rendered from `packages/opencode/opencode.base.json`; edit there, not in the live file | Hides the LAN-capable browser's tools. Version-specific: `disabled`/`enabled` did not work on 2.0.29; re-test `tools` vs `permissions` on upgrade |
| skills | both | AI | `test -f ~/.config/opencode/skills/<name>/SKILL.md` for each skill | `./setup.sh configure --only opencode` installs managed skills; symlink `dev-den-setup` separately | This skill included |
| notifier plugin | both | CMD | `jq -e '.plugins[] \| select(startswith("@mohak34/opencode-notifier"))' ~/.config/opencode/opencode.json` | add to plugins | Optional |
| desktop pet plugin | desktop only | — | — | — | Do not install on the laptop |

The installer adds `~/.opencode/bin` to `.bashrc` only. On a zsh login shell, add
it to `~/.zshenv`; otherwise `opencode` disappears after switching shells.

The desktop app does not inherit the shell PATH. If `opencode mcp list` reports
`NotFound: npx` for `figma` or `chrome-devtools`, or a self-signed error for
`obsidian`, see `docs/opencode-desktop.md` for the launcher wrapper, desktop
override, and verification. A config reload does not change the service
environment; fully restart the app.

`img2threejs` is installed per machine and is not shipped by the dotfiles repo; its
absence in `verify` is a warning, not a failure.

## 10. MCP servers

MCP servers are rendered into `opencode.json` under `mcp.servers` from
`~/projects/dotfiles/packages/opencode/opencode.base.json` plus
`~/.config/dev-den/machine.json`. Check the entry, then test the server itself.
`opencode mcp list` reports the running service, not the file, so it does not
prove the configuration.

| Server | Applies to | Mode | Check | Install | Needs |
| --- | --- | --- | --- | --- | --- |
| obsidian | both | GUI + AI | authenticated `GET /vault/` (section 11); the remote MCP is `https://127.0.0.1:27124/mcp/` | Rendered from dotfiles; enter the token | Obsidian running, plugin enabled, token file, plugin cert trusted |
| chrome-devtools | both | CMD | `curl -fsS http://127.0.0.1:9222/json/version` | launch the browser with a debugging port (see below) | Brave |
| obscura | both | AI | `obscura --version` | pinned binary from the release page, verify its SHA-256 | — |
| obscura-dev | both | AI | `obscura --version` | same binary, same entry, hidden by the `tools` glob | obscura |
| chatgpt-bridge | both | AI | `~/projects/chatgpt-bridge/.venv/bin/chatgpt-bridge-mcp` exists and runs | clone the project and create its venv (see section 12) | docker, python3 |
| computer-use-linux | both | AI | `computer-use-linux doctor \| jq -e '.readiness.blockers == []'` | prebuilt release binaries (verify SHA-256) into `~/.local/bin`, then `computer-use-linux setup` | — |
| figma | both | AI | `FIGMA_API_KEY` file present and non-empty | `npx -y figma-developer-mcp --stdio` | Figma token |
| proxmox | both | AI + CMD | the token check in section 13 returns `200` | see section 13 | uv, Proxmox token |
| comfy | desktop only | — | — | — | GPU |
| blender | desktop only | CMD | `blender --version` | install Blender, enable its MCP add-on | GPU recommended |
| unity | desktop only | — | — | — | GPU, Unity Editor |

`computer-use-linux` drives the **real** desktop (AT-SPI trees, KWin windows,
uinput/portal input, portal screenshots). It is registered **globally** in
`opencode.json`; `setup` enables AT-SPI (a per-user gsettings toggle), the first
screenshot prompts for portal consent, and WebKitGTK apps expose only their
top-level window (not the internal DOM). Runbook:
`Tools/Computer-Use-Linux` in the Engineering vault.

`agent-workspace-linux` is the **project-local** sibling (a hidden desktop for
native GUI QA), not in the global `opencode.json` table above: it is registered
per project in that project's `.opencode/opencode.jsonc`. The prebuilt binary
`~/.local/bin/agent-workspace-linux` (v0.6.4, SHA-256 verified) needs
`xorg-x11-server-Xvfb openbox xdotool` for the X11 backend (KWin needs none).
Check: `agent-workspace-linux doctor` reports `ready_for_x11_workspace` and
`ready_for_kwin_workspace` true. Runbook: `Tools/Agent-Workspace-Linux` in the
Engineering vault.

Brave debugging launch (for chrome-devtools). `/usr/bin/brave-browser` is the
confirmed Fedora binary. To make CDP the default, install a wrapper and point the
user `.desktop` entry at it:

```sh
#!/bin/sh
exec /usr/bin/brave-browser --remote-debugging-port=9222 --remote-debugging-address=127.0.0.1 "$@"
```

Put it at `~/.local/bin/brave-browser`, then override
`~/.local/share/applications/brave-browser.desktop` to use that absolute path.
Close any running Brave first, or the flag is ignored. Verify with the check above.

## 11. Obsidian and engineering memory

| Item | Applies to | Mode | Check | Install | Notes |
| --- | --- | --- | --- | --- | --- |
| Obsidian | both | AI + GUI | `flatpak info md.obsidian.Obsidian`, or `test -x ~/AppImages/obsidian.appimage` and confirm the process/API | Flatpak, or the AppImage from <https://obsidian.md> | This laptop uses `~/AppImages/obsidian.appimage`, not Flatpak. Launch it once |
| vault folder | both | AI | `test -d ~/Documents/Obsidian/Engineering` | Fast Note Sync syncs it from the homelab server | Sync must finish first. Do not touch the vault before then |
| vault sync | both | GUI | Fast Note Sync shows this device as paired and idle | pair with the homelab server (URL in the private note) | Pairing is per device |
| `obsidian-local-rest-api` | both | GUI | `grep -q '"obsidian-local-rest-api"' ~/Documents/Obsidian/Engineering/.obsidian/community-plugins.json` | enable in Settings → Community plugins | — |
| `fast-note-sync` | both | GUI | same pattern | enable and pair | — |
| `run-in-background` | both | GUI | same pattern | enable; Obsidian starts at login | Keeps the REST endpoint available |
| REST API reachable | both | AI | authenticated `GET /vault/` returns `200`; the same call without the token returns `401` | — | `/` returns `200` unauthenticated, so it is not a check. Use a mode-600 header file, not argv. Trust the plugin cert with `--cacert` or `NODE_EXTRA_CA_CERTS` |

The remote MCP is HTTPS with the plugin's self-signed certificate. Extract
`.crypto.cert` from the plugin `data.json` and trust it through
`NODE_EXTRA_CA_CERTS`; the exact command and launcher are in
`docs/opencode-desktop.md`. Do not disable verification.

## 12. chatgpt-bridge

| Item | Applies to | Mode | Check | Install | Notes |
| --- | --- | --- | --- | --- | --- |
| project clone | both | AI | `test -d ~/projects/chatgpt-bridge/.git` | `git clone <chatgpt-bridge repository> ~/projects/chatgpt-bridge` | Private repository: needs `gh auth` first |
| venv + entry point | both | AI | `test -x ~/projects/chatgpt-bridge/.venv/bin/chatgpt-bridge-mcp` | `python3 -m venv .venv` then `.venv/bin/pip install -e .` | — |
| browser backend | both | CMD | `chatgpt-bridge doctor` reports the browser reachable | Use the existing Brave CDP endpoint (`CDP_URL=http://127.0.0.1:9222`), or `chatgpt-bridge browser start` for container mode | This laptop uses Brave CDP, not the Docker container |
| browser login | both | GUI | `chatgpt-bridge doctor` reports the sidebar/chats readable | Sign in once in whichever browser is used | Per machine |

## 13. Proxmox MCP

Reference: `RekklesNA/ProxmoxMCP-Plus` (`https://github.com/RekklesNA/ProxmoxMCP-Plus`), pinned to
`0.6.2`, run over stdio with `uvx proxmox-mcp-plus==0.6.2`. Configuration keys, their
sources, and the role and token steps are in `docs/proxmox-mcp.md`.

Do not check the MCP by starting it: it is a stdio server and waits for a client.
Check the token against the Proxmox API instead (see the doc for the command).

| Step | Applies to | Mode | Check | Notes |
| --- | --- | --- | --- | --- |
| Private env file | both | AI | `test -s <PRIVATE_ENV_FILE>` and mode `600` | Holds `PROXMOX_*` and `REQUESTS_CA_BUNDLE`; sources the token file |
| Token works | both | AI | API version call returns `200` (command in the doc) | Proves the token and host, without starting the MCP |
| MCP launches | both | AI | `command -v uvx` succeeds | Full launch is tested by OpenCode, not by the skill |
| Role and token creation | both | CMD | verified on the homelab, see the doc | Run only after reading each output |

Read-only is the default. Write privileges are added only for a documented operation
the user approves.

The MCP child reads its environment when it is spawned. After rotating the token,
restart the OpenCode backend and re-run `opencode mcp list`; a config reload alone
keeps the old secret in the child.

Access model: the API token handles the control plane; guest work uses
per-container SSH as root with `~/.ssh/id_homelab` (one key per machine), installed
into every guest from the host with `pct exec`. The host shell is break-glass only.
The MCP's `cwd` is the private config directory, so its `proxmox-jobs.sqlite3` stays
out of repositories. If the homelab repo is present,
`scripts/check-permissions.sh` checks the token's effective privilege set in one
read-only call.

## 14. Local AI (desktop only)

| Item | Applies to | Mode | Check | Install | Reference |
| --- | --- | --- | --- | --- | --- |
| Ollama | desktop | AI | `ollama --version` | official installer | — |
| models | desktop | AI | `ollama list` | `ollama pull <model>` | qwen3.5 9b, gemma4 9b, gemma4 26b |
| ComfyUI service | desktop | AI | `systemctl --user is-active comfyui` prints `active` | private setup note | GPU |
| NVIDIA driver | desktop | AI | `nvidia-smi` | system package | RTX 50-series needs CUDA 12.8+ builds |

## 15. Homelab (remote, configuration only)

These are not installed here. The skill only checks that this machine can reach
them. Connection details are in the private note.

| Item | Applies to | Mode | Check |
| --- | --- | --- | --- |
| Proxmox host | both | AI | version endpoint answers |
| Fast Note Sync server | both | AI | server answers |
| Coolify | both | AI | `coolify` context answers, token file present |
| Tailscale client | both | CMD | `tailscale status` |

---

## Dependencies at a glance

```
Obsidian MCP  ← Obsidian ← vault synced ← Fast Note Sync paired
              ← REST plugin enabled ← REST token file
              ← opencode config (merged) ← opencode
chatgpt-bridge ← docker ← docker access ← (new login)
              ← python3, git, gh auth
Proxmox MCP   ← uv/uvx ← proxmox token ← role ← Proxmox host reachable
Figma MCP     ← npx (Node via fnm) ← figma token file
computer-use-linux ← prebuilt binary + AT-SPI enabled
agent-workspace-linux ← Xvfb + openbox + xdotool (X11 backend)
```
