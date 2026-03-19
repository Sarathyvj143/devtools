---
description: Export local memories to plugin repo for cross-machine sync
argument-hint: [all | <memory-name> | <type>]
allowed-tools: [Read, Write, Glob, Grep, Bash]
---

# Memory Export

$ARGUMENTS

## Instructions

Export local memories into the devtools plugin repo (`memories/` directory) so they sync across machines.

1. **Find the plugin repo path** -- locate the devtools plugin installation:
   - Check `~/.claude/plugins/installed_plugins.json` for the devtools install path
   - The `memories/` directory is at `<install-path>/memories/`

2. **Scan local memories:**
   - User-level: `~/.claude/memory/*.md`
   - Project-level: `~/.claude/projects/<current-project-hash>/memory/*.md`

3. **Filter based on $ARGUMENTS:**
   - `all` -- export everything found
   - A specific name -- export only that memory
   - A type (user/feedback/project/reference) -- export all of that type
   - No argument -- list all found memories and ask the user which to export

4. **For each selected memory:**
   - Read the memory file
   - Ensure it has the required frontmatter (name, description, type, captured, last-reviewed)
   - Copy to `<plugin-repo>/memories/<type>/<name>.md`
   - Add entry to `<plugin-repo>/memories/MEMORY.md` index

5. **Remind the user** to commit and push the plugin repo to sync across machines:
   ```
   Exported N memories to devtools plugin repo.
   To sync across machines:
     cd <plugin-repo-path>
     git add memories/
     git commit -m "chore: export memories"
     git push
   ```
