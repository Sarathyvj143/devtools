# DevTools — Gemini CLI Context

You have devtools installed. This file is the Gemini bootstrap for the plugin — the equivalent of Claude Code's session-start hook. It is loaded on every session where the devtools extension is active.

## Available Components

| Path | Purpose | Status on Gemini (v1) |
|---|---|---|
| `skills/` | Workflow skills (auto-triggered via `activate_skill` from each skill's `description`). | **Active.** Gemini auto-discovers from `skills/` inside this extension. |
| `agents/` | Specialized subagents with project-context placeholders. | **Not invokable in v1** — base templates exist but no Gemini agent dispatcher is wired. Deferred to v2b. |
| `commands/` | Slash commands (e.g. `/remember`, `/orchestrate`). Authored as Markdown for Claude Code. | **Not installed in v1** — Gemini requires TOML format. Deferred to v2a. On Claude Code: available. |
| `hooks/` | Lifecycle event triggers. | **Not active on Gemini.** Gemini has no `SessionStart` hook equivalent — that's why you're reading this file instead. |
| `memories/` | Portable cross-machine memories. | **File sync works**, but `/memory-export` and `/memory-import` commands are not installed on Gemini in v1. On Claude Code: available. |

## Tool Name Mapping

This plugin uses Claude Code tool names by default. On Gemini CLI, see `skills/using-devtools/references/gemini-tools.md` for the full mapping (e.g. `Read → read_file`, `Bash → run_shell_command`, `Skill → activate_skill`).

## The `using-devtools` Meta-Skill

A meta-skill named `using-devtools` ships with this plugin. Gemini will inject its name + description into the system prompt automatically at session start; activate it (`activate_skill using-devtools`) for the full overview of components, mappings, and memory ops.

## Auto-Memory Detection

Watch for these patterns during conversation:

- If the user **corrects your approach**, offer to save as a *feedback* memory.
- If you **discover a project convention** (naming, layout, conventions), offer to save as a *project* memory.
- If the user states a **tool or style preference**, offer to save as a *user* memory.
- If you **solve a tricky bug**, offer to save the root cause as a *reference* memory.

Always ask before saving. Format: *"Want me to remember this for future sessions?"*

On Gemini, persistence uses the `save_memory` tool (writes to `GEMINI.md`). On Claude Code, persistence uses the file-based memory system under `~/.claude/memory/`. Both are valid storage backends; the auto-detection patterns are identical.

## Platform-Specific Notes

- **No `SessionStart` hook on Gemini.** This file is the substitute. Anything that depended on dynamic checks (e.g. the Claude Code "plugin updated, run /assemble-team --update" notice) is unavailable on Gemini in v1.
- **Slash commands** (`/remember`, `/memory-export`, `/orchestrate`, `/assemble-team`) are Claude-Code-only in v1. Gemini TOML versions are deferred to v2a.
- **Subagents** in `agents/` are not invokable on Gemini in v1. Deferred to v2b.
- **Skills auto-trigger from their `description` field** — same as Claude Code. Description quality is the single biggest factor for usefulness.

## Updating

There is no `/plugin update` on Gemini. To pull the latest plugin contents:

```bash
cd <your-extension-install-dir> && git pull
```

See `GEMINI-INSTALL.md` (added in Phase 3) for the full install + update procedure.
