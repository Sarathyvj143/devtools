---
name: memory-sync
description: Use this skill when the user wants to sync, export, or import memories across machines — keywords like "sync memories", "export memories", "import memories", "share memories across machines", "load memories on a new machine", "back up memories", "pull memories from the plugin repo". Cross-platform replacement for the Claude-Code-only /memory-export and /memory-import commands; works identically on Claude Code, Codex, and Gemini CLI.
---

# Memory Sync — Portable Cross-Platform

This skill is the platform-agnostic replacement for `/memory-export` and `/memory-import`. The slash commands work only on Claude Code; this skill works wherever skills load (Claude Code, Codex, Gemini CLI). Use it when a user wants to move memories between local storage and the portable `memories/` directory inside the devtools plugin/extension repo.

The behavior here is **byte-equivalent on Claude Code** to running `/memory-export` and `/memory-import`. On Codex and Gemini, this skill is the only way to perform these operations.

## Determine intent

Read the user's request and classify:

- **Export** — user wants to push local memories into the plugin repo (e.g. "save these memories to sync", "back up my memories", "export to plugin").
- **Import** — user wants to pull memories from the plugin repo into local storage (e.g. "load memories on this machine", "import the team memories").
- **Two-way sync** — user wants both directions. Run export first, then import; ask for confirmation between the two stages.

If intent is ambiguous, ask the user once which direction they want.

## Locate the plugin/extension path

The plugin install location varies by platform. Resolve in this order, stopping at the first match:

| Platform | Lookup |
|---|---|
| Claude Code | Read `~/.claude/plugins/installed_plugins.json`; find the entry whose key contains `devtools`; use its `installPath`. |
| Codex | Default convention: `~/.codex/devtools` (per `.codex/INSTALL.md`). |
| Gemini CLI | Default convention: `~/.gemini/extensions/devtools` (per `GEMINI-INSTALL.md`). Skills-only-install users may have it at the symlink target — ask if neither default exists. |

If none of the conventional paths exist, ask the user for the plugin/extension repo path. The directory is recognized by containing a `memories/` subdirectory plus `skills/`, `commands/`, `agents/`.

The portable memory directory is `<plugin-root>/memories/<type>/<name>.md`.

## Locate local memory directories

Local memory tiers vary by platform. Scan all that apply:

| Platform | User-tier (global) | Project-tier |
|---|---|---|
| Claude Code | `~/.claude/memory/` | `~/.claude/projects/<project-hash>/memory/` |
| Codex | `~/.agents/memory/` (convention; may not exist if user hasn't created memories yet) | `<project-root>/.agents/memory/` (convention) |
| Gemini CLI | `~/.gemini/memory/` (or content stored via the `save_memory` tool inside `GEMINI.md`) | `<project-root>/.gemini/memory/` (convention) |

Only Claude Code has a documented project-hash scheme. On other platforms, project memories live next to the project; if absent, treat as "no project memories on this platform."

## Memory file format (consistent across platforms)

Each memory file is markdown with required frontmatter:

```markdown
---
name: short-kebab-case-name
description: One-line description
type: <user|feedback|project|reference>
captured: YYYY-MM-DD
source-project: <project name>
last-reviewed: YYYY-MM-DD
---

<Memory content>

**Why:** <reason>
**How to apply:** <when and how>
```

If a memory lacks required frontmatter, ask the user to fill it in before exporting/importing.

## Export flow

1. **Scan local memories** in all applicable tiers for the current platform.
2. **Filter by user request** — `all`, a specific `name`, a specific `type` (`user`/`feedback`/`project`/`reference`), or list available and ask which to export.
3. **Validate frontmatter** — every selected memory must have `name`, `description`, `type`, `captured`, `last-reviewed`. Skip any that fail validation, with a warning.
4. **For each valid memory:**
   - Read the file content.
   - Write to `<plugin-root>/memories/<type>/<name>.md`. Create the type subdirectory if missing.
   - Append an entry to `<plugin-root>/memories/MEMORY.md` index. The entry format is `- [<name>](<type>/<name>.md) — <description>`.
5. **Report a summary**: count exported per type; any skipped with reason.
6. **Remind the user** to commit and push the plugin repo:

```bash
cd <plugin-root>
git add memories/
git commit -m "chore: export memories"
git push
```

## Import flow

1. **Scan portable memories** at `<plugin-root>/memories/<type>/*.md`. Skip `.gitkeep` and the top-level `MEMORY.md` index.
2. **Filter by user request** — `all`, a specific `name`, a specific `type`, or list available and ask which to import.
3. **For each selected memory, route to the correct local tier by type:**
   - `user`, `feedback`, `reference` → user-tier (global) for the current platform.
   - `project` → project-tier for the current platform.
4. **Detect duplicates.** If a memory with the same `name` already exists at the target, ask the user whether to overwrite, skip, or rename.
5. **Write the file** to the target path.
6. **Update the local `MEMORY.md`** index at the destination to register the new entry.
7. **Report a summary**: count imported per type; any skipped with reason.

## Two-way sync flow

1. Run export. Report results.
2. Pause. Confirm with user before continuing.
3. Run import. Report results.

## Platform-specific tool notes

| Tool the model needs | Claude Code | Codex | Gemini |
|---|---|---|---|
| Read a file | `Read` | `Shell` `cat` (or `ReadFile` if available) | `read_file` |
| Write a file | `Write` | `Shell` with `> ` redirect (or `WriteFile`) | `write_file` |
| List directory | `Glob` | `Shell` `ls` | `list_directory` |
| Search content | `Grep` | `Shell` `grep` | `grep_search` |
| Git operations | `Bash git ...` | `Shell` `git ...` | `run_shell_command` `git ...` |

See `skills/using-devtools/references/codex-tools.md` and `skills/using-devtools/references/gemini-tools.md` for the full mapping.

## Behavior parity check (for the model)

When this skill is invoked on Claude Code with the same arguments that would have been passed to `/memory-export` or `/memory-import`, the resulting filesystem changes must be **byte-equivalent** to running those commands. Specifically:
- Same source paths scanned.
- Same destination paths written.
- Same `MEMORY.md` index entry format.
- Same duplicate-handling prompts.

If a Claude Code user invokes this skill instead of the slash command, behavior is identical. The slash commands remain available; this skill does not replace them on Claude Code, only complements them with cross-platform access.
