# Decisions

Dated record of what was chosen and why. Newest first.

## 2026-11-02: one skill and one inventory, not a framework

**Chosen.** One OpenCode skill (`skills/dev-den-setup`) and one inventory file. The
AI inspects the machine, reports status, and runs or guides each step.

**Rejected.** A Bash provisioning framework with phases, a template renderer for
the OpenCode config, and per-package verify scripts. It was complicated to
maintain, and each new tool meant more code.

**Why.** The AI can run and read commands itself. The hard part is knowing what
each item is and how to check it, which is knowledge, not code. Knowledge fits in
a Markdown file and a skill.

## 2026-11-02: three execution modes

- `AI`: the AI can inspect, create, edit, install, and verify.
- `CMD`: the AI prints the exact command. I run it in my terminal. Used for
  interactive logins, secrets, sudo, and anything that changes a remote system.
- `GUI`: no CLI or API exists. The AI gives numbered steps and waits for `done`.

A command-doable step is never turned into a GUI walkthrough.

## 2026-11-02: statuses are proven by checks

`installed`, `configured`, `working`, `missing`, `unknown`. A directory existing,
or a config entry existing, does not count as working.

## 2026-11-02: public procedures, private values

The repository is written so it can be made public. It holds procedures, check
commands, and placeholders. Real hostnames, IP addresses, application IDs, domain
names, tokens, and key paths stay in a private note on each machine and in
`~/.config/secrets/`.

## 2026-11-02: Proxmox MCP

Reference implementation: `RekklesNA/ProxmoxMCP-Plus`, version `0.6.2`, run with
`uvx`. Read-only by default. Write privileges are added only for a documented
operation that I approve.

## 2026-11-02: Obsidian initial sync

The existing homelab Fast Note Sync server is authoritative for a new machine.
Sync must finish before any tool touches the vault.

## 2026-11-02: versions

Not pinned. The desktop versions are the reference, so the setup is known to work.
Otherwise the latest stable version is used.

## 2026-11-02: the desktop is covered too

The same skill inventories the desktop and the laptop. Checking the desktop first
proves the inventory matches a machine that already works.
