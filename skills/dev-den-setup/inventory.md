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
| `env` file | both | CMD | `test -s ~/.config/secrets/env` | You write it (see below) | Sourced by `~/.zshenv` |
| Figma token | both | CMD | `test -s ~/.config/secrets/figma-token` | `printf '%s' 'TOKEN' > ~/.config/secrets/figma-token && chmod 600 ~/.config/secrets/figma-token` | Raw file, no trailing newline |
| Obsidian REST token | both | GUI | `test -s ~/.config/secrets/obsidian-token` | Copy from the plugin settings, then `printf '%s' 'KEY' > …` | Generated per install. Never copy from another machine |
| Proxmox token | both | CMD | `test -s ~/.config/secrets/proxmox-token` | See section 9 | Raw file, no trailing newline |
| homelab SSH key | both | CMD | `test -f ~/.ssh/<homelab-key>` | `ssh-keygen -t ed25519 -f ~/.ssh/<homelab-key> -C "<machine-name>"` | One key per machine. Add the public key to the homelab yourself |

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
| Docker CE | both | AI | `docker --version` | Docker's Fedora repository, then `sudo dnf install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin` | Docker 29.x |
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
| lazygit | both | AI | `lazygit --version` | `sudo dnf install -y lazygit` | — |
| tmux | both | AI | `tmux -V` | `sudo dnf install -y tmux` | — |
| stylua | both | AI | `stylua --version` | `cargo install stylua` or the upstream release | Neovim formatter |
| pre-commit | both | AI | `pre-commit --version` | `uv tool install pre-commit` | — |
| Coolify CLI | both | CMD | `coolify --version` | Install from the Coolify CLI release page, put it in `/usr/local/bin` | Desktop 1.8.0. Context points at the homelab, in the private note |
| Cloudflare `cf` CLI (beta) | both | AI | `cf --version` | `npm install -g cf` | Beta. Install only. Do not log in as part of setup |
| codex CLI | desktop | CMD | `codex --version` | Install from its release page | Optional |

## 7. Shell

| Item | Applies to | Mode | Check | Install | Notes |
| --- | --- | --- | --- | --- | --- |
| zsh | both | AI | `zsh --version` | `sudo dnf install -y zsh` | — |
| default shell is zsh | both | CMD | `getent passwd "$USER" \| grep -q zsh` | `chsh -s "$(command -v zsh)"` | Needs a new login |
| oh-my-zsh | both | AI | `test -d ~/.oh-my-zsh` | official installer, unattended | — |
| zsh-autosuggestions | both | AI | `test -d ~/.oh-my-zsh/custom/plugins/zsh-autosuggestions` | `git clone https://github.com/zsh-users/zsh-autosuggestions ~/.oh-my-zsh/custom/plugins/zsh-autosuggestions` | — |
| `~/.zshenv` sources secrets | both | AI | `grep -q 'config/secrets/env' ~/.zshenv` | Add the three-line guard: source the file only if it exists | Check the presence of the line. Never print the file |
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
| OpenCode CLI | both | AI | `opencode --version` | `curl -fsSL https://opencode.ai/install \| bash` | 2.0.29 |
| global config | both | AI | `jq empty ~/.config/opencode/opencode.json` | merge, do not overwrite (see SKILL.md) | — |
| global AGENTS.md | both | AI | `test -s ~/.config/opencode/AGENTS.md` | copy from the dotfiles repository | — |
| `tools` glob for LAN browser | desktop + laptop | AI | `jq -e '.tools["obscura-dev_*"] == false' ~/.config/opencode/opencode.json` | merge the key | Hides the LAN-capable browser's tools. Entry-level `disabled` and `enabled` keys do not work on 2.0.29 |
| skills | both | AI | `test -f ~/.config/opencode/skills/<name>/SKILL.md` for each skill | copy or symlink from the repository | This skill included |
| notifier plugin | both | CMD | `jq -e '.plugins[] \| select(startswith("@mohak34/opencode-notifier"))' ~/.config/opencode/opencode.json` | add to plugins | Optional |
| desktop pet plugin | desktop only | — | — | — | Do not install on the laptop |

## 10. MCP servers

Each server is listed in `opencode.json` under `mcp.servers`. Check the entry, then
test the server itself. `opencode mcp list` reports the running service, not the
file, so it does not prove the configuration.

| Server | Applies to | Mode | Check | Install | Needs |
| --- | --- | --- | --- | --- | --- |
| obsidian | both | GUI + AI | authenticated `curl` to the local REST API (see section 11) | AI writes the entry, you enter the token | Obsidian running, plugin enabled, token file |
| chrome-devtools | both | CMD | `curl -fsS http://127.0.0.1:9222/json/version` | launch the browser with a debugging port (see below) | Brave |
| obscura | both | AI | `obscura --version` | pinned binary from the release page, verify its SHA-256 | — |
| obscura-dev | both | AI | `obscura --version` | same binary, same entry, hidden by the `tools` glob | obscura |
| chatgpt-bridge | both | AI | `~/projects/chatgpt-bridge/.venv/bin/chatgpt-bridge-mcp` exists and runs | clone the project and create its venv (see section 12) | docker, python3 |
| figma | both | AI | `FIGMA_API_KEY` file present and non-empty | `npx -y figma-developer-mcp --stdio` | Figma token |
| proxmox | both | AI + CMD | `uvx proxmox-mcp-plus==0.6.2 --help` runs, and the read-only check in section 13 passes | see section 13 | uv, Proxmox token |
| comfy | desktop only | — | — | — | GPU |
| blender | desktop only | CMD | `blender --version` | install Blender, enable its MCP add-on | GPU recommended |
| unity | desktop only | — | — | — | GPU, Unity Editor |

Brave debugging launch (command-doable, for chrome-devtools). The binary name is
unverified on Fedora; confirm it with `command -v brave-browser` before use:

```sh
brave-browser --remote-debugging-port=9222 --remote-debugging-address=127.0.0.1
```

Close any running Brave first, or the flag is ignored. Verify with the check above.
Record the confirmed binary name here once it is known.

## 11. Obsidian and engineering memory

| Item | Applies to | Mode | Check | Install | Notes |
| --- | --- | --- | --- | --- | --- |
| Obsidian | both | AI + GUI | `flatpak info md.obsidian.Obsidian` | `flatpak install -y flathub md.obsidian.Obsidian` | Flatpak. Launch it once |
| vault folder | both | AI | `test -d ~/Documents/Obsidian/Engineering` | Fast Note Sync syncs it from the homelab server | Sync must finish first. Do not touch the vault before then |
| vault sync | both | GUI | Fast Note Sync shows this device as paired and idle | pair with the homelab server (URL in the private note) | Pairing is per device |
| `obsidian-local-rest-api` | both | GUI | `grep -q '"obsidian-local-rest-api"' ~/Documents/Obsidian/Engineering/.obsidian/community-plugins.json` | enable in Settings → Community plugins | — |
| `fast-note-sync` | both | GUI | same pattern | enable and pair | — |
| `run-in-background` | both | GUI | same pattern | enable; Obsidian starts at login | Keeps the REST endpoint available |
| REST API reachable | both | AI | `curl -fsSk -H "Authorization: Bearer $(cat ~/.config/secrets/obsidian-token)" https://127.0.0.1:27124/` returns 200 | — | Certificate is self-signed, so `-k` is expected |

## 12. chatgpt-bridge

| Item | Applies to | Mode | Check | Install | Notes |
| --- | --- | --- | --- | --- | --- |
| project clone | both | AI | `test -d ~/projects/chatgpt-bridge/.git` | `git clone <chatgpt-bridge repository> ~/projects/chatgpt-bridge` | Private repository: needs `gh auth` first |
| venv + entry point | both | AI | `test -x ~/projects/chatgpt-bridge/.venv/bin/chatgpt-bridge-mcp` | `python3 -m venv .venv` then `.venv/bin/pip install -e .` | — |
| browser container | both | CMD | `docker ps --format '{{.Names}}' \| grep -qx chatgpt-bridge-browser` | `chatgpt-bridge browser start` | — |
| browser login | both | GUI | `chatgpt-bridge doctor` passes | open the browser UI, sign in to ChatGPT once | Per machine |

## 13. Proxmox MCP

Reference: `RekklesNA/ProxmoxMCP-Plus` at `https://github.com/RekklesNA/ProxmoxMCP-Plus`,
pinned to `0.6.2`, launched with `uvx`. Every command in this section has the status
noted in `docs/proxmox-mcp.md`. Read that before running any of them.

| Step | Applies to | Mode | Check | Notes |
| --- | --- | --- | --- | --- |
| Proxmox API endpoint | both | AI | HTTPS GET of the version endpoint returns 200 | URL in the private note |
| Proxmox role | both | CMD | `pveum role list` contains the role name | Privilege list in `docs/proxmox-mcp.md` |
| Proxmox user + token | both | CMD | `pveum user token list <user>` lists the token | Token secret is shown once. Capture it into the secrets file, do not print it |
| env file for the MCP | both | AI | `test -s <env file>` and it has no trailing newline in the token | Private path. Mode 600 |
| MCP connectivity | both | AI | `uvx proxmox-mcp-plus==0.6.2` starts and a read-only tool returns data | Read-only first |

Read-only is the default. Write privileges are added only for a documented
operation, and I approve each one.

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
```
