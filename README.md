# Dev Den

A single OpenCode skill that knows my development environment, reports what is
already set up on the current machine, and sets up whatever I choose.

It owns the workflow and the checks. The configuration files it installs
(`opencode.json`, `AGENTS.md`, managed skills, nvim/tmux/zsh) still come from
`~/projects/dotfiles`, rendered from `~/.config/dev-den/machine.json`. The AI does
the work it can do itself. For anything that needs me, it prints the exact command
or instructions, waits for me to say `done`, re-checks, and continues.

## Contents

| Path | What it is |
| --- | --- |
| `skills/dev-den-setup/SKILL.md` | The skill: workflow, status rules, pause-and-wait rule, backup rule |
| `skills/dev-den-setup/inventory.md` | Every item: applicability, check, install, mode, version reference |
| `docs/decisions.md` | Architecture decisions and why they were made |
| `docs/proxmox-mcp.md` | Proxmox MCP config, token rotation, and safe checks |
| `docs/opencode-desktop.md` | Desktop launcher PATH, Obsidian CA trust, and verification |

## Install the skill

```sh
git clone https://github.com/eydamson-dev/dev-den.git ~/projects/dev-den
mkdir -p ~/.config/opencode/skills
ln -sfn ~/projects/dev-den/skills/dev-den-setup ~/.config/opencode/skills/dev-den-setup
```

The skill is only the inventory. The configuration files come from
`~/projects/dotfiles`; run `./setup.sh configure` there to install `opencode.json`,
`AGENTS.md`, the managed skills, and the nvim/tmux/zsh packages. Use
`--only opencode` to restrict it to OpenCode.

The repository is public, so the clone needs no credentials. If it is ever made
private again, authenticate first with
`gh auth login --hostname github.com --git-protocol https --web`.

An SSH clone works too if the machine has a key registered with GitHub:
`git clone git@github.com:eydamson-dev/dev-den.git ~/projects/dev-den`.

A symlink means that `git pull` in `~/projects/dev-den` updates the skill
everywhere it is linked. Restart OpenCode after installing it, so the skill is
discovered.

Check that it was found: in OpenCode, ask "what skills do you have?" and confirm
that `dev-den-setup` appears.

## How to use it

Start OpenCode in any directory and ask in plain language. Examples:

| Ask | What happens |
| --- | --- |
| "Run the dev-den inventory" | Detects this machine, checks every applicable item, and shows a status table |
| "What is missing on this machine?" | The same inventory, filtered to `missing` and `unknown` |
| "Set up the CLI tools" | Checks the CLI group, shows the commands, and installs what you choose |
| "Set up the Proxmox MCP" | Walks through its steps. Commands are printed for you to run |
| "Check the Obsidian MCP" | Runs only the Obsidian checks |
| "Show me the dev list" | Lists every item and its applicability, without checking anything |

### What the AI does on its own

- Reads the machine (OS, architecture, RAM, GPU) and decides whether it is a
  desktop or a laptop.
- Runs read-only checks and reports one status per item:
  `installed`, `configured`, `working`, `missing`, or `unknown`.
- Installs packages and edits config, after backing the file up first.

### What it asks you to do

For some steps the AI cannot act alone. There are two kinds:

- **Command steps:** the AI prints a command in a code block, for example a
  login or a token creation. Copy it, run it in your terminal, then reply `done`.
- **GUI steps:** the AI prints numbered instructions, for example enabling an
  Obsidian plugin. Follow them, then reply `done`.

After you reply `done`, the AI re-runs the check before it continues. It trusts
the check, not your word.

### Secrets

The AI never prints or reads a secret value. Secrets live in `~/.config/secrets/`
with mode `600`; a file is either a single raw value with no trailing newline or a
labelled key file such as `TOKEN_NAME`/`SECRET`. The AI checks that a file exists
and has the right mode, and compares hashes when a value must be compared.

### Machine-specific values

Hostnames, IP addresses, application IDs, and similar values are not in this
repository. Keep them in `~/.config/dev-den/` — `machine.json` for role and feature
flags, `local/AGENTS.md` for machine instructions, and `local/infra.md` for hosts
and IDs. The AI reads those when a step needs a value. If they are missing, the AI
asks you for the value and tells you where to record it.

## Test it on a machine

See [`docs/testing.md`](docs/testing.md) for step-by-step testing, including what
to expect at each stage.

## Public repository

This repository is public. It holds procedures and placeholders only. Real
hostnames, IP addresses, application IDs, domain names, tokens, and key paths
never go in it. The Proxmox env file, secret files, and machine notes stay on
each machine.

A commit that publishes such a value stays published in the git history even
after it is removed, so check before committing.

## Status

Verified on the laptop on 2026-11-03: every applicable inventory item is installed,
configured, or working. The Proxmox token, CA, and MCP steps are verified; see
`docs/proxmox-mcp.md`.
