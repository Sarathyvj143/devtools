---
description: Import portable memories from plugin repo into local Claude memory
argument-hint: [all | <memory-name> | <type>]
allowed-tools: [Read, Write, Glob, Grep, Bash]
---

# Memory Import

$ARGUMENTS

## Instructions

Import memories from the devtools plugin repo into local Claude memory on this machine.

1. **Find the plugin repo path** — locate the devtools plugin installation:
   - Check `~/.claude/plugins/installed_plugins.json` for the devtools install path
   - The `memories/` directory is at `<install-path>/memories/`

2. **Scan portable memories:**
   - Read all `.md` files in `<plugin-repo>/memories/user/`, `feedback/`, `project/`, `reference/`
   - Skip `.gitkeep` files and `MEMORY.md`

3. **Filter based on $ARGUMENTS:**
   - `all` — import everything
   - A specific name — import only that memory
   - A type — import all of that type
   - No argument — list all available and ask the user which to import

4. **Route each memory by type** (read from frontmatter):
   - `user` or `feedback` → write to `~/.claude/memory/` (global)
   - `project` → write to `~/.claude/projects/<current-project-hash>/memory/` (project-specific)
   - `reference` → write to `~/.claude/memory/` (global)

5. **Check for duplicates** — if a memory with the same `name` already exists at the target location, ask the user whether to overwrite or skip.

6. **Update MEMORY.md** index at each target location.

7. **Confirm** — tell the user how many memories were imported and where.
