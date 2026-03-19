---
description: Manually save something to memory for future sessions
argument-hint: <what to remember>
allowed-tools: [Read, Write, Glob, Grep, Bash]
---

# Remember

$ARGUMENTS

## Instructions

The user wants to save something to memory. Follow these steps:

1. **Parse the input** -- understand what the user wants remembered from $ARGUMENTS

2. **Determine memory type** -- classify as one of:
   - `user` -- personal preference, tool choice, coding style
   - `feedback` -- correction to your behavior, "don't do X", "always do Y"
   - `project` -- project-specific convention, architecture pattern, quirk
   - `reference` -- external resource pointer, debugging solution, useful link

   If the type is ambiguous, ask the user which type fits best.

3. **Generate a memory file** with this format:

   ```markdown
   ---
   name: short-kebab-case-name
   description: One-line description of what this memory is about
   type: <user|feedback|project|reference>
   captured: <today's date YYYY-MM-DD>
   source-project: <current project name>
   last-reviewed: <today's date YYYY-MM-DD>
   ---

   <Memory content>

   **Why:** <reason this matters>
   **How to apply:** <when and how to use this memory>
   ```

4. **Save locally** based on type:
   - `user` or `feedback` -> write to `~/.claude/memory/` (global, user-level)
   - `project` -> write to `~/.claude/projects/<current-project-hash>/memory/` (project-specific)
   - `reference` -> write to `~/.claude/memory/` (global, user-level)

5. **Update MEMORY.md** -- add an entry to the `MEMORY.md` index at the save location

6. **Confirm** -- tell the user what was saved and where. Mention they can run `/memory-export` to sync it to the plugin repo for cross-machine access.
