# Proxmox MCP

Reference: `RekklesNA/ProxmoxMCP-Plus`, `https://github.com/RekklesNA/ProxmoxMCP-Plus`,
pinned to version `0.6.2`. OpenCode runs it over stdio with `uvx`.

## Status legend

Each claim is marked with its source:

- `upstream`: stated in the ProxmoxMCP-Plus README (read on 2026-11-03).
- `proxmox`: standard Proxmox API behaviour, not from upstream.
- `verified`: exercised against the homelab on 2026-11-03.
- `untested`: not yet run on a real host. Do not run it without reading the output first.

## Configuration (upstream)

The server reads configuration from environment variables, or from a JSON file
named by `PROXMOX_MCP_CONFIG`. Environment variables are the simpler option for
OpenCode, because the secrets stay in a private file and never enter the config.

| Variable | Purpose | Status |
| --- | --- | --- |
| `PROXMOX_HOST` | Proxmox host name or address (private note) | upstream |
| `PROXMOX_PORT` | API port, normally `8006` | upstream |
| `PROXMOX_USER` | User the token belongs to, for example `<user>@pve` | upstream |
| `PROXMOX_TOKEN_NAME` | Token ID, the part after `!` | upstream |
| `PROXMOX_TOKEN_VALUE` | Token secret. Never printed, never committed | upstream |
| `PROXMOX_VERIFY_SSL` | `true` in normal use. Certificate checks stay on | upstream |
| `PROXMOX_MCP_CONFIG` | Optional path to a JSON config file instead of variables | upstream |

Transport: stdio, launched by OpenCode as `uvx proxmox-mcp-plus==0.6.2`.
Stdio does not need `MCP_API_KEY`. That key applies only to the HTTP and OpenAPI
modes, which this setup does not use. `upstream`

## Principles

- Read-only first. Add write privileges only for an operation the user approves.
- Use a separate API token with privilege separation enabled. `proxmox`
- The secret is stored in a private env file with mode 600. It is never printed
  in chat, never written into this repository, and never placed on a command line
  where shell history records it.
- The MCP's documentation says a token must have only the permissions its
  workflows need. `upstream`

## Verifying the token without starting the MCP

Starting the MCP launches a stdio server, which waits for a client. So do not use
it as a test. Test the token directly against the API instead. The secret is read
from the private env file into a mode-600 header file, never into argv — a
malformed curl command echoes its arguments back. Only the status code prints.

```sh
( set -a; . <PRIVATE_ENV_FILE>; set +a
  header=$(mktemp)
  chmod 600 "$header"
  printf 'Authorization: PVEAPIToken=%s!%s=%s\n' \
    "$PROXMOX_USER" "$PROXMOX_TOKEN_NAME" "$PROXMOX_TOKEN_VALUE" > "$header"
  curl -fsS --cacert "${REQUESTS_CA_BUNDLE:?set REQUESTS_CA_BUNDLE to the PVE CA bundle}" \
    -o /dev/null -w '%{http_code}\n' -H @"$header" \
    "https://${PROXMOX_HOST}:${PROXMOX_PORT:-8006}/api2/json/version"
  rm -f "$header" )
```

Expected output: `200`. `proxmox` (the `PVEAPIToken` header format is the standard
Proxmox API token format). TLS stays verified: the private env file points
`REQUESTS_CA_BUNDLE` at a bundle containing the PVE CA. Do not use `-k` as a fix
for a TLS error.

## Creating the role and token (verified on the homelab)

The upstream documentation does not describe creating roles or tokens. These steps
were run on the homelab on 2026-11-03; the concrete role name and privilege set
stay in the private note, not this repository. Run them only after reading the
output of each one.

| # | Step | Status | Command shape |
| --- | --- | --- | --- |
| 1 | Check the API answers | verified | the `curl` check above, without the token |
| 2 | Create a role with the privileges the workflows need | verified | `pveum role add <ROLE> -privs "<PRIVS>"` |
| 3 | Create the user | verified | `pveum user add <USER>@pve` |
| 4 | Grant the role on the paths it needs | verified | `pveum aclmod <PATH> -user <USER>@pve -role <ROLE>` |
| 5 | Create the token with privilege separation | verified | `pveum user token add <USER>@pve <TOKEN_ID> --privsep 1` |
| 6 | Save the secret without printing it | verified | redirect step 5's output into the private file, do not display it |
| 7 | Verify with the `curl` check above | verified | expect `200` |

Notes:

- Step 5 prints the secret once. Capture it, do not display it.
- Choose privileges from the upstream tool list, not from memory. Start read-only.
- Privilege separation means the token gets only what the ACL grants it. If the
  token is refused an action, check the user's ACL before widening the role.

## Rotating the secret

A token secret cannot be read back. Rotate it in place and capture the new value
without printing it:

```sh
pveum user token modify <USER>@<REALM> <TOKEN_ID> --regenerate 1 --output-format json
```

The command returns `value` once. Write it to the private secret file, then
**restart the OpenCode backend** — an MCP child inherits its environment when it is
spawned, so a config reload re-spawns it with the old secret. Verify with a real
API call (above), then `opencode mcp list`.

If the homelab repo is present, `scripts/check-permissions.sh` verifies the token's
effective privilege set in one read-only call. A difference is a decision to make,
not a surprise to absorb.

## Lessons carried over from the homelab

Kept in the Engineering vault, not here. The general points:

- Token privilege separation is an intersection: the token gets only the rights
  that both the user and the token have. A missing ACL on the user blocks the token.
- Creating a usable guest can need an extra privilege beyond the obvious VM ones.
- Upstream READMEs often suggest turning privilege separation off. Don't, unless
  you understand the consequence.

Specific role and privilege names belong in the private note, not here.
