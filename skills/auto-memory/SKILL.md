---
name: auto-memory
description: Use when you detect a memory-worthy moment (user correction, preference, project discovery, debug solution) to offer saving it
---

# Auto-Memory Capture

This skill is injected into sessions via the SessionStart hook. It guides you to detect and offer to save memory-worthy moments.

## Detection Patterns

Watch for these during conversation:

| Pattern | Memory Type | Example |
|---------|------------|---------|
| User corrects your approach | `feedback` | "No, don't mock the database" |
| User states a preference | `user` | "I always use pnpm" |
| You discover a project convention | `project` | "This repo uses barrel exports" |
| A tricky bug is resolved | `reference` | "The timeout was caused by..." |

## Process

1. **Detect** -- recognize the pattern from the table above
2. **Confirm** -- ask: "Want me to remember this for future sessions?"
3. **On confirmation:**
   - Determine memory type from the pattern
   - Generate a memory file with proper frontmatter:
     ```
     name: short-kebab-case-name
     description: one-line summary
     type: user|feedback|project|reference
     captured: YYYY-MM-DD
     source-project: current project name
     last-reviewed: YYYY-MM-DD
     ```
   - Save to appropriate local path:
     - `user`/`feedback`/`reference` -> `~/.claude/memory/`
     - `project` -> `~/.claude/projects/<current-project-hash>/memory/`
   - Update `MEMORY.md` index
   - Mention `/memory-export` for cross-machine sync

## Key Principles

- **Never save without asking** -- always confirm first
- **One memory per moment** -- don't batch multiple learnings
- **Be specific** -- descriptions should be clear enough to decide relevance later
