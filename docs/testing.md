# Testing Dev Den

Test in this order. Stop at the first step that fails and report what you saw.

The first two phases are **read-only**. They change nothing on the machine.

## Phase 0: Install the skill (the only change)

The repository is private, so authenticate first:

```sh
gh auth login --hostname github.com --git-protocol https --web
```

Then install the skill:

```sh
git clone https://github.com/eydamson-dev/dev-den.git ~/projects/dev-den
mkdir -p ~/.config/opencode/skills
ln -sfn ~/projects/dev-den/skills/dev-den-setup ~/.config/opencode/skills/dev-den-setup
ls -l ~/.config/opencode/skills/dev-den-setup/SKILL.md
```

Expected: the last command shows the file, with no error.

Restart OpenCode. Then ask: **"what skills do you have?"**

Expected: `dev-den-setup` is in the list. If not, check the symlink target with
`readlink -f ~/.config/opencode/skills/dev-den-setup`.

## Phase 1: Desktop inventory (read-only)

Ask OpenCode: **"Run the dev-den inventory."**

Expected:

- It detects this machine as **desktop** (NVIDIA GPU present).
- It prints a table with one status per item. Items you know are installed, such
  as git, docker, opencode, and gh, should show `installed` or `working`.
- It does not install, change, or delete anything.
- It does not print any secret value.

Check three things yourself:

1. Pick two items you know are missing or optional, such as starship, and confirm
   the table says `missing`.
2. Pick one item you know is working, such as `docker info`, and confirm it says
   `working`, not just `installed`.
3. Look at the MCP section. An item shows `configured` or `working` only if the
   check was actually run.

Record any row that is wrong. A wrong status on the desktop is a bug in the
inventory, and the laptop run would repeat it.

## Phase 2: Desktop, one read-only check per area

Ask one at a time, and read the answers:

- "Check the Obsidian MCP." Expected: the REST API check passes, and no token is
  printed.
- "Check the chatgpt-bridge MCP." Expected: the entry point is found and runs.
- "Check the Docker daemon." Expected: `working`.
- "Show me the dev list." Expected: a list of every item and its applicability,
  with no checks run.

## Phase 3: Something that changes a file (backup check)

Only do this on the desktop, and only after Phase 1 is correct.

Ask: **"Check the tmux TPM item. Do not install it."**

Expected: the AI reports the status and does not change anything. Then confirm
that no file in `~/.config/` was modified:

```sh
find ~/.config -newer ~/projects/dev-den/README.md -type f 2>/dev/null | head
```

If a file changed, check that it has a `.bak-<timestamp>` copy next to it. The
skill must not change a file without making that copy first.

## Phase 4: A command step (pause-and-wait)

Pick an item that is `missing` and low risk. For example, `ripgrep` is probably
installed already, so choose something you do not have.

Ask: **"Set up <item>."**

Expected:

1. The AI shows the exact command, and says whether it needs `sudo`.
2. It waits. It does not run the command itself unless the item is `AI` mode.
3. You run the command yourself, reply `done`, and the AI re-runs the check.
4. The status changes to `installed` or `working`. If it does not, the AI says so
   and does not claim success.

## Phase 5: The laptop (later)

Run the same phases on the laptop. Start with the skill install and Phase 1. Do
not start installing anything until the inventory reads correctly on the laptop.
The laptop has no GPU, so GPU items should be skipped, not reported as missing.

## What a failure looks like

| What you see | What it means | What to report |
| --- | --- | --- |
| The skill is not listed | The symlink or the restart did not work | The output of `readlink -f` |
| The AI runs an install without asking | The workflow rule was not followed | The exact prompt and reply |
| A status says `working` but the tool is broken | The check is too weak | The item name and the command that failed |
| A secret value appears in the chat | A secret leaked | Stop, and rotate the secret |
| It changes a file with no backup | The backup rule was not followed | The file name |

## Undo the test install

```sh
rm ~/.config/opencode/skills/dev-den-setup
```

This removes only the symlink. It does not touch the repository.
