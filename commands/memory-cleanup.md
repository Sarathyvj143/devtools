---
description: Review and prune stale memories
argument-hint: [--threshold <days>]
allowed-tools: [Read, Write, Glob, Grep, Bash]
---

# Memory Cleanup

$ARGUMENTS

## Instructions

Review accumulated memories and help the user prune stale or outdated ones.

1. **Parse threshold** -- default is 90 days. If $ARGUMENTS contains `--threshold <N>`, use N days instead.

2. **Scan all memory locations:**
   - Local user-level: `~/.claude/memory/*.md`
   - Local project-level: `~/.claude/projects/<current-project-hash>/memory/*.md`
   - Portable: find devtools plugin path via `~/.claude/plugins/installed_plugins.json`, then scan `<path>/memories/**/*.md`

3. **For each memory, extract from frontmatter:**
   - `name`
   - `type`
   - `captured` date
   - `last-reviewed` date
   - `description`

4. **Flag memories** where `last-reviewed` (or `captured` if no `last-reviewed`) is older than the threshold.

5. **Present to user** in a grouped list:

   ```
   ## Potentially Stale Memories (older than 90 days)

   ### User Preferences
   - prefer-pnpm (captured: 2025-12-01) -- "User prefers pnpm over npm"

   ### Feedback
   - no-mocking-db (captured: 2025-11-15) -- "Don't mock the database in tests"

   ## Recent Memories (keeping)
   - use-vitest (captured: 2026-02-28) -- "Use vitest instead of jest"
   ```

6. **Ask the user** which stale memories to remove. Accept:
   - "remove all stale"
   - specific names: "remove prefer-pnpm and no-mocking-db"
   - "keep all" to skip

7. **For each removed memory:**
   - Delete the file from local storage
   - Delete from portable storage (if it exists there)
   - Remove entry from relevant `MEMORY.md` index

8. **Update `last-reviewed`** on kept memories to today's date.
