# Proxmox MCP

Reference: `RekklesNA/ProxmoxMCP-Plus`, `https://github.com/RekklesNA/ProxmoxMCP-Plus`,
version `0.6.2`, run with `uvx proxmox-mcp-plus==0.6.2`.

## Status of this document

Every command here is **untested** until it is checked against the upstream
README for 0.6.2 and run on a real Proxmox host. Each step is marked:

- `verified`: checked against the upstream documentation for this version.
- `untested`: not yet checked. Do not run it without reading the output first.

## Principles

- Read-only first. Add write privileges only for an operation I approve.
- Separate identity and token. Privilege separation on the token.
- The token secret is captured into `~/.config/secrets/proxmox-token` with mode
  600. It is never printed in chat, never written to this repository, and never
  placed on a command line where shell history records it.
- The MCP reads its configuration from a private environment file, referenced by
  path. The file is not in this repository.

## Steps

| # | Step | Status | Command shape |
| --- | --- | --- | --- |
| 1 | Confirm the API endpoint answers | untested | `curl -fsSk https://<PROXMOX_HOST>:8006/api2/json/version` |
| 2 | Create the role with read privileges | untested | `pveum role add <ROLE> -privs "<PRIVS>"` |
| 3 | Create the user | untested | `pveum user add <USER>@pve` |
| 4 | Grant the role on the paths it needs | untested | `pveum aclmod / -user <USER>@pve -role <ROLE>` |
| 5 | Create the token with privilege separation | untested | `pveum user token add <USER>@pve <TOKEN_ID> --privsep 1` |
| 6 | Save the secret without echoing it | untested | run step 5 with output redirected into the secrets file, do not print it |
| 7 | Start the MCP and make one read-only call | untested | `uvx proxmox-mcp-plus==0.6.2` with the environment file loaded |

Notes on the table:

- Step 5 prints the token secret once. Capture it, do not show it.
- Step 2 privileges come from the upstream documentation. Copy the list from there,
  do not write it from memory.
- Privilege separation means the token has only what the ACL grants it. If the
  token cannot do something, check step 4 before widening the role.

## Lessons carried over from the homelab

Recorded in the Engineering vault (`Lessons/`), not in this repository:

- Token privilege separation is an intersection: the token gets only the rights that
  both the user and the token have. A missing ACL on the user blocks the token.
- Creating a usable guest needs an SDN use privilege, not only the VM privileges.
- Upstream READMEs often say to disable privilege separation. Do not follow that
  without understanding the consequence.

These lessons are general enough to keep. The specific role and privilege names
belong in the private note.
