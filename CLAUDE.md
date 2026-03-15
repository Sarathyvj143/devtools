# DevTools — Project Conventions

## What This Repo Is

Personal portable Claude Code plugin. Contains skills, agents, commands, hooks, and memory management.

## Directory Layout

- `skills/<name>/SKILL.md` — workflow skills with YAML frontmatter
- `agents/<name>.md` — subagent definitions with YAML frontmatter
- `commands/<name>.md` — slash commands with YAML frontmatter
- `hooks/` — lifecycle event hooks (bash scripts + JSON config)
- `memories/` — portable cross-machine memories
- `.claude-plugin/` — plugin and marketplace metadata
- `.codex/` — Codex platform support

## Adding Components

Copy the `_template` file in each directory as your starting point.

## Rules

- Keep skills focused — one clear purpose per skill
- Descriptions must be specific — they control when skills trigger
- Test on Claude Code before pushing (primary platform)
- Version bump `.claude-plugin/plugin.json` on meaningful changes
- Never commit secrets or `.env` files
- Hook scripts must output valid JSON (see hooks/session-start for format)
