# DevTools

Personal portable toolkit for AI-assisted development.
Skills, agents, commands, and hooks — one install, every machine.

## Installation

### Claude Code

```bash
# Add the marketplace
/plugin marketplace add your-username/devtools

# Install the plugin
/plugin install devtools@devtools-marketplace
```

### Codex

See [.codex/INSTALL.md](.codex/INSTALL.md)

## What's Inside

| Directory | Purpose | How to Add |
|-----------|---------|-----------|
| `skills/` | Workflow skills (auto-triggered) | Copy `_template/`, rename, edit `SKILL.md` |
| `agents/` | Specialized subagents | Copy `_template.md`, rename, edit |
| `commands/` | Slash commands (`/my-cmd`) | Copy `_template.md`, rename, edit |
| `hooks/` | Lifecycle event triggers | Edit `hooks.json` + add scripts |
| `memories/` | Portable cross-machine memories | Use `/remember`, `/memory-export` |
| `.mcp.json` | MCP server config template | Edit, copy to project root |

## Memory Commands

| Command | Purpose |
|---------|---------|
| `/remember <text>` | Manually save something to memory |
| `/memory-export` | Export local memories to plugin repo for sync |
| `/memory-import` | Import plugin repo memories on a new machine |
| `/memory-cleanup` | Review and prune stale memories |

Memories are also auto-detected during sessions (corrections, preferences, discoveries, debug solutions). You'll be asked before anything is saved.

## Updating

```bash
/plugin update devtools
```

## Adding a New Skill

1. `cp -r skills/_template skills/my-skill`
2. Edit `skills/my-skill/SKILL.md`
3. Commit and push
4. `/plugin update devtools` on each machine

Same pattern for agents (`agents/`) and commands (`commands/`).

## Cross-Platform Support

| Platform | Status | Config |
|----------|--------|--------|
| Claude Code | Active | `.claude-plugin/` |
| Codex | Active | `.codex/INSTALL.md` |
| Cursor | Placeholder | `.cursor-plugin/` (fill when needed) |
| Gemini CLI | Placeholder | `GEMINI.md` (fill when needed) |
| OpenCode | Placeholder | `.opencode/` (fill when needed) |
