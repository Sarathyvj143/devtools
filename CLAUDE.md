# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

Personal portable Claude Code plugin (v1.1.0). Configuration-based (markdown + JSON) -- no build/test/lint infrastructure. Contains skills, agents, commands, hooks, and memory management.

## Commands

There is no build, lint, or test runner -- every component is markdown + JSON, validated at session-start by the plugin's own hooks. Common operations are slash commands run from a *consuming* project, not from this repo:

- `/plugin marketplace add Sarathyvj143/devtools` then `/plugin install devtools@devtools-marketplace` -- install
- `/plugin update devtools` -- pull latest after editing & pushing here
- `/assemble-team` -- audit team in current project (no writes)
- `/assemble-team --update` -- selective regen, keeps project context
- `/assemble-team --regenerate` -- full rebuild from scratch
- `/orchestrate "<task>"` -- run the 6-phase multi-agent workflow
- `/remember`, `/memory-export`, `/memory-import`, `/memory-cleanup` -- memory ops

Add a new component: `cp -r skills/_template skills/my-skill` (or `commands/_template.md`, `agents/_template.md`), edit, commit, push, then `/plugin update devtools` on each machine.

### Local validation (pre-commit)

No CI -- run these manually before pushing:

```powershell
Get-Content hooks\hooks.json | ConvertFrom-Json   # validate hook JSON syntax
git diff --check                                  # whitespace errors
```

For JSON profiles under `agents/_profiles/`, the same `ConvertFrom-Json` check applies.

Commit style: Conventional Commits (`feat:`, `fix:`, `refactor:`) with a short imperative subject.

## Architecture

### Component Types and Frontmatter

Every skill, agent, and command is a Markdown file with YAML frontmatter that controls behavior:

- **Skills** (`skills/<name>/SKILL.md`): `name`, `description` -- description controls auto-triggering
- **Agents** (`agents/<name>.md`): `name`, `description`, `model` (sonnet|inherit), `allowed-tools` list
- **Commands** (`commands/<name>.md`): `description`, `argument-hint`, `allowed-tools` list

### Agent System (Key Complexity)

The agent system has three layers:

1. **Base templates** (`agents/_bases/`, 26 files) -- reusable agent templates with `{{PLACEHOLDER}}` variables (e.g. `{{PROJECT_NAME}}`, `{{TECH_STACK}}`, `{{STARTUP_COMMANDS}}`)
2. **Service profiles** (`agents/_profiles/services/`, 17 files) -- JSON detection profiles (React, Node, Python, Go, Docker, etc.) that map file patterns to tech stacks, ports, and commands
3. **Composition profiles** (`agents/_profiles/compositions/`, 4 files) -- JSON templates defining which agents to generate for common project shapes (fullstack-react-node, fullstack-react-python, microservices, general)

`/assemble-team` detects a project's services, matches profiles, substitutes variables into base templates, and writes generated agents to the target project's `.claude/agents/` + `.claude/team-config.json`.

### Agent Update Flow

Plugin changes propagate to generated agents via a version-tracking mechanism:
1. `session-start` hook compares plugin commit hash against `team-config.json`'s recorded commit
2. If mismatched, prompts user to run `/assemble-team --update`
3. Update uses per-agent template hash comparison for selective regeneration while preserving project-specific context (services, paths, ports)

### Hooks

`hooks/hooks.json` registers a single `SessionStart` hook on `startup|resume|clear|compact`, which runs `hooks/run-hook.cmd session-start`. That script injects the `using-devtools` SKILL.md plus an auto-memory detection block, and emits the plugin-update notice when `team-config.json` is stale.

`hooks/run-hook.cmd` is a polyglot batch/bash wrapper -- batch sees a cmd block, bash sees it as a heredoc no-op. Requires Git for Windows (Git Bash) on Windows; hooks silently disable without it. Hook scripts must output JSON with `hookSpecificOutput.additionalContext`.

### Orchestrator (6-Phase Workflow)

The `/orchestrate` command runs a phased multi-agent pipeline: Discovery → Design → Planning → Implementation → Verification → Completion. Each phase has specific agents, output files, and gate conditions. All artifacts land in the *consuming project's* `.claude/orchestrator/runs/YYYY-MM-DD-<slug>/` -- not this repo.

### Memory System

4 types (`user`, `feedback`, `project`, `reference`), each with frontmatter including `captured` date and `source-project`. User/feedback/reference memories are global (`~/.claude/memory/`); project memories are project-scoped. Portable sync via `/memory-export` and `/memory-import` through the `memories/` directory.

## Platform Support

- **Claude Code** (primary, active) -- `.claude-plugin/plugin.json`
- **Codex** (active, skills only) -- `.codex/INSTALL.md`; tool mappings expanded beyond the original 6-row table in `skills/using-devtools/references/codex-tools.md`. Real-machine regression check is **deferred** (see `docs/gemini-verification-log.md`).
- **Gemini CLI** (active, skills only — v1) -- `GEMINI-INSTALL.md`; bootstrap via `GEMINI.md` (Gemini has no `SessionStart` hook); tool mappings in `skills/using-devtools/references/gemini-tools.md`. Static validation passed; real-machine verification on Gemini CLI is **deferred** (see `docs/gemini-verification-log.md`). Commands and agents are not yet ported (deferred to v2a / v2b).
- **Cursor / OpenCode** -- stub configs only (`.cursor-plugin/`, `.opencode/`). They do not currently install anything; treat them as placeholders, not documented integrations.

`AGENTS.md` (Codex/generic) is a parallel agent-instruction file for non-Claude-Code tools. `GEMINI.md` is the Gemini CLI extension's `contextFileName` -- it serves as the bootstrap that Gemini loads in place of a `SessionStart` hook. Keep all three in rough sync, but `CLAUDE.md` is authoritative for Claude Code.

`.mcp.json` at the repo root is a template for MCP server config; keep it portable, not personal.

## Reference

- `docs/superpowers/specs/` -- design docs for the plugin and the orchestrator
- `docs/superpowers/plans/` -- corresponding implementation plans
- `tests/` -- empty placeholder; no automated tests exist

## Rules

- Copy `_template` files as starting points for new components
- Keep skills focused -- one clear purpose per skill
- Descriptions must be specific -- they control when skills trigger
- Test on Claude Code before pushing (primary platform)
- Version bump `.claude-plugin/plugin.json` on meaningful changes
- Never commit secrets or `.env` files
- Filenames: lowercase-hyphen (`memory-cleanup.md`, not `MemoryCleanup.md`)
- JSON: two-space indent, stable key ordering
- Generated runtime instructions must use **absolute paths** -- a `cd` into a subdirectory must not break log/artifact paths (see commit `0da0985` for the precedent)
- Hook scripts must be cross-platform aware -- `hooks/run-hook.cmd` is the polyglot batch/bash model to follow
