---
name: dev-den-setup
description: Inventory and set up my development environment on this machine (Fedora desktop or laptop). Use when I ask to check, list, install, or verify my dev setup, the OpenCode config, MCP servers, the Obsidian vault connection, CLI tools, shell, or the Proxmox MCP. Reports the status of each item, shows the list, lets me pick a subset, and walks me through steps that need me.
---

# Dev Den setup

## Goal

Know every item in `inventory.md`, check the actual state of this machine, report
it honestly, and set up only what I choose. The AI does what it can do itself.
When a step needs me, it gives me the exact command or precise instructions,
waits for me to say `done`, runs the item's check, and only then continues.

## Workflow

1. **Discover the machine.** Read `/etc/os-release`, `uname -m`, RAM, and the GPU
   (`nvidia-smi -L` or `lspci`). Decide the role: `desktop` if an NVIDIA GPU with
   compute is present and the private machine note says so, otherwise `laptop`.
   Read the private machine note if it exists. Never assume a role from GPU
   detection alone.
2. **Inventory.** For every item in `inventory.md` that applies to this role, run
   its check and assign one status (see below). Show the result as a table.
3. **Select.** Show the full list of available items with their status. Ask which
   to set up: all, the items marked `missing` or `unknown`, or a subset I name.
4. **Execute, one item at a time.** Respect dependencies: an item's `needs` must
   be `working` before it starts.
   - `AI-doable`: run it, then run its check.
   - `command-doable`: print the exact command in a code block, say what it does,
     and wait. Do not turn it into a web UI walkthrough when a command exists.
   - `GUI-only`: print numbered instructions and wait.
   - When I say `done`, run the check. Do not take my word for it.
5. **Recheck and report.** Re-run the checks for everything touched and show the
   final status table.

## Status rules

| Status | Meaning | Proof required |
| --- | --- | --- |
| `installed` | The program exists and reports a version | `command -v` plus a version command |
| `configured` | Required config exists and is structurally valid | Parse it (for example `jq empty`) and confirm the expected keys |
| `working` | A real functional test passes | A command that talks to the service or runs the tool |
| `missing` | Confirmed absent | The check ran and found nothing |
| `unknown` | The check could not decide | Say why, and what I need to decide it |

A directory existing is not proof. A config entry is not proof the server runs.
Report the strongest status the evidence supports, and no stronger.

## Rules for changing files

- Before changing an existing file, copy it to `<name>.bak-<YYYYmmdd-HHMMSS>`.
- Merge into existing config. Never replace a file wholesale if it has content I
  might want.
- After a change, validate the file. If validation fails, restore the backup and
  say so.
- Leave alone anything not in the inventory: runtime state, auth files, pairing
  files, and plugin state directories.

## Secrets

- Values live in `~/.config/secrets/`, permissions `600`, directory `700`. A file
  is either a single raw value with no trailing newline, or a labelled key file
  such as `TOKEN_NAME`/`SECRET` or `URL`/`TOKEN`.
- Never print or read a secret value in chat, in a command output, or in a file in
  this repository. Check that a secret file exists and has the right mode. If a
  value must be compared, compare a hash. Do not `cat` it.
- Never put a secret in argv. Pass a header through a mode-600 file and remove the
  file after use.
- After changing a secret or an env file, restart the OpenCode backend. MCP
  children read their environment when they are spawned.
- If a secret is missing, tell me the file path and how to create it. Do not ask
  me to paste it into the chat.

## Machine-specific values

Hostnames, IP addresses, application IDs, container IDs, and similar facts are not
in this repository. They live in `~/.config/dev-den/`: `machine.json` for the role
and feature flags, `local/AGENTS.md` for this machine's instructions, and
`local/infra.md` for hosts and IDs. Read those when a step needs one. If a value is
missing, ask me for it and tell me where to record it. Do not invent one.

## Safety

- Never make a machine-wide change without showing me the command first.
- Never run destructive commands (deleting data, resetting Docker, overwriting
  credentials) without an explicit yes from me for that exact action.
- Commands needing `sudo` are shown to me first.
- Prefer read-only checks. Do not write to a remote system (including the
  Proxmox host) unless I asked for that exact change.

## Interaction

- Ask one or two sharp questions when something is ambiguous. Do not guess.
- When I say I am away, keep going on items that need no input, skip the ones
  that do, and summarize at the end.
- Keep the status table current as items change.
