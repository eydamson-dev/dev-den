# Dev Den

A single OpenCode skill that knows my development environment, reports what is
already set up on the current machine, and sets up whatever I choose.

It replaces a larger provisioning framework. The AI does the work it can do
itself. For anything that needs me, it prints the exact command or instructions,
waits for me to say `done`, re-checks, and continues.

## Contents

| Path | What it is |
| --- | --- |
| `skills/dev-den-setup/SKILL.md` | The skill: workflow, status rules, pause-and-wait rule, backup rule |
| `skills/dev-den-setup/inventory.md` | Every item: applicability, check, install, mode, version reference |
| `docs/decisions.md` | Architecture decisions and why they were made |
| `docs/proxmox-mcp.md` | Proxmox MCP setup steps (marked untested until verified) |

## Install the skill

The repository is **private**. Authenticate first, once per machine:

```sh
gh auth login --hostname github.com --git-protocol https --web
```

Then install the skill:

```sh
git clone https://github.com/eydamson-dev/dev-den.git ~/projects/dev-den
mkdir -p ~/.config/opencode/skills
ln -sfn ~/projects/dev-den/skills/dev-den-setup ~/.config/opencode/skills/dev-den-setup
```

An SSH clone works too if the machine has a key registered with GitHub:
`git clone git@github.com:eydamson-dev/dev-den.git ~/projects/dev-den`.

`gh auth login` also stores a credential helper, so a later `git pull` works
without asking again.

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

The AI never prints a secret value. Secrets live in `~/.config/secrets/` as raw
files with no trailing newline and mode `600`. The AI checks that a file exists
and is non-empty. It does not read the contents.

### Machine-specific values

Hostnames, IP addresses, application IDs, and similar values are not in this
repository. Keep them in a private note on each machine. The AI reads that note
when a step needs a value. If the note is missing, the AI asks you for the value
and tells you where to record it.

## Test it on a machine

See [`docs/testing.md`](docs/testing.md) for step-by-step testing, including what
to expect at each stage.

## Status

Draft. The inventory and the skill are being verified step by step against the
desktop before the laptop run. Items marked `untested` in `docs/proxmox-mcp.md`
have not been run yet.
