# DevTools — Personal Portable Plugin

**Date:** 2026-03-14
**Status:** Approved
**Approach:** Single Plugin Repo (Plugin + Self-Hosted Marketplace)

---

## 1. Overview

A personal portable toolkit packaged as a Claude Code plugin that installs with a single command on any machine. Contains skills, agents, slash commands, hooks, and MCP server configs. Starts as a skeleton with templates — components are added over time.

### Goals
- One-command install across 4-6 machines
- Works with Claude Code and Codex (extensible to Cursor, Gemini CLI, OpenCode later)
- Skeleton/template approach — ready to fill, not pre-loaded with content
- Single GitHub repo serves as both plugin and marketplace

### Non-Goals
- Not a team/org or public marketplace (personal use)
- Not pre-populated with domain-specific skills
- No bootstrap scripts or external dependency setup

---

## 2. Repository Structure

```
devtools/
├── .claude-plugin/
│   ├── plugin.json              # Plugin identity + metadata
│   └── marketplace.json         # Self-hosted marketplace manifest
├── .codex/
│   └── INSTALL.md               # Codex installation guide
├── .cursor-plugin/              # Future — Cursor support (includes component pointers)
│   └── plugin.json
├── .opencode/                   # Future — OpenCode support
│   └── INSTALL.md
├── gemini-extension.json        # Future — Gemini CLI extension manifest
├── GEMINI.md                    # Future — Gemini CLI tool mapping
├── .gitignore                   # Repo hygiene
├── .gitattributes               # Line ending normalization
├── memories/                    # Portable memories (cross-machine sync)
│   ├── MEMORY.md               # Index of portable memories
│   ├── user/                   # User preferences
│   ├── feedback/               # Corrections and guidance
│   ├── project/                # Cross-project discoveries
│   └── reference/              # Debug solutions, external pointers
├── skills/
│   ├── _template/
│   │   └── SKILL.md             # Starter template for new skills
│   ├── auto-memory/
│   │   └── SKILL.md             # Auto-capture detection skill
│   └── using-devtools/
│       ├── SKILL.md             # Meta-skill: platform tool mapping
│       └── references/
│           ├── codex-tools.md   # Claude Code → Codex tool names
│           └── gemini-tools.md  # Claude Code → Gemini tool names
├── agents/
│   └── _template.md             # Starter template for new agents
├── commands/
│   ├── _template.md             # Starter template for new commands
│   ├── remember.md              # Manual memory capture (/remember)
│   ├── memory-export.md         # Export local memories → plugin repo
│   ├── memory-import.md         # Import plugin repo → local machine
│   └── memory-cleanup.md        # Review and prune stale memories
├── hooks/
│   ├── hooks.json               # Hook event registrations
│   ├── session-start            # Bash hook script
│   └── run-hook.cmd             # Windows polyglot wrapper
├── .mcp.json                    # MCP server config template (copy to project)
├── tests/                       # Future — skill and integration tests
├── README.md                    # Usage + installation docs
└── CLAUDE.md                    # Project conventions
```

### Future platform directories
- `.cursor-plugin/` — requires explicit component pointers in `plugin.json` (unlike Claude Code's convention-based discovery)
- `.opencode/`, `GEMINI.md`, `gemini-extension.json` — created as empty placeholders, activated when adopting that platform
- `references/` folder inside `using-devtools` skill maps tool names between platforms
- `gemini-extension.json` format: `{"name": "devtools", "description": "...", "version": "1.0.0", "contextFileName": "GEMINI.md"}`

---

## 3. Plugin Configuration

### `.claude-plugin/plugin.json`

Claude Code discovers components by convention (scanning `skills/`, `agents/`, `commands/`, `hooks/hooks.json`), so this file contains only identity and metadata — no component pointers.

```json
{
  "name": "devtools",
  "description": "Personal portable toolkit — skills, agents, commands, and hooks across all projects",
  "version": "1.0.0",
  "author": { "name": "your-name" },
  "homepage": "https://github.com/your-username/devtools",
  "repository": "https://github.com/your-username/devtools",
  "license": "MIT",
  "keywords": ["toolkit", "skills", "agents", "portable", "personal"]
}
```

### `.cursor-plugin/plugin.json` (Future — Cursor needs explicit pointers)

```json
{
  "name": "devtools",
  "displayName": "DevTools",
  "description": "Personal portable toolkit — skills, agents, commands, and hooks across all projects",
  "version": "1.0.0",
  "author": { "name": "your-name" },
  "license": "MIT",
  "skills": "./skills/",
  "agents": "./agents/",
  "commands": "./commands/",
  "hooks": "./hooks/hooks.json"
}
```

### `.claude-plugin/marketplace.json`

```json
{
  "name": "devtools-marketplace",
  "owner": { "name": "your-name" },
  "plugins": [
    {
      "name": "devtools",
      "source": "./",
      "description": "Personal portable toolkit",
      "version": "1.0.0",
      "author": { "name": "your-name" }
    }
  ]
}
```

The repo is both a plugin (installable) and a marketplace (discoverable).

---

## 4. Installation Flow

### Claude Code (primary)

```bash
# One-time setup on any machine
/plugin marketplace add your-username/devtools
/plugin install devtools@devtools-marketplace

# Update across machines
/plugin update devtools
```

### Codex

See `.codex/INSTALL.md`:
1. Clone repo: `git clone https://github.com/your-username/devtools.git ~/.codex/devtools`
2. Symlink skills: `mkdir -p ~/.agents/skills && ln -s ~/.codex/devtools/skills ~/.agents/skills/devtools`
   > **Note:** Verify symlink path against current Codex docs — path may vary by version.
3. Reference tool mapping in `skills/using-devtools/references/codex-tools.md`

### Auto-configure per project (optional)

Add `.claude/settings.json` to any project:
```json
{
  "extraKnownMarketplaces": {
    "devtools-marketplace": {
      "source": { "source": "github", "repo": "your-username/devtools" }
    }
  },
  "enabledPlugins": {
    "devtools@devtools-marketplace": true
  }
}
```

---

## 5. Component Templates

### Skill Template (`skills/_template/SKILL.md`)

```markdown
---
name: skill-name
description: Use when the user asks to... (specific trigger conditions)
---

# Skill Name

## When to Use
- Trigger conditions

## Process
1. Step one
2. Step two

## Key Principles
- Principle one
```

### Agent Template (`agents/_template.md`)

```markdown
---
name: agent-name
description: Specialized agent for... (3-5 words)
model: sonnet
allowed-tools: [Read, Glob, Grep, Bash]
---

# Agent Name

You are a specialized agent that...

## Your Task
- What you do

## Rules
- Constraints and guardrails
```

### Command Template (`commands/_template.md`)

```markdown
---
description: Short description shown in /help
argument-hint: <required-arg> [optional-arg]
allowed-tools: [Read, Glob, Grep, Bash]
---

# Command Name

$ARGUMENTS

## Instructions
What to do with the arguments
```

---

## 6. Hooks

### `hooks/hooks.json`

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup|resume|clear|compact",
        "hooks": [
          {
            "type": "command",
            "command": "\"${CLAUDE_PLUGIN_ROOT}/hooks/run-hook.cmd\" session-start",
            "async": false
          }
        ]
      }
    ]
  }
}
```

- `session-start` bash script runs on every session start
- `run-hook.cmd` is a Windows polyglot wrapper (detects platform, finds bash on Windows via Git for Windows paths) — copied from the superpowers reference implementation
- Hooks can be extended with additional events: `PreToolUse`, `PostToolUse`, `Notification`, `Stop`, `SubagentStop`

### Hook Output Contract

Hook scripts must output JSON in this format:

**Claude Code:**
```json
{"hookSpecificOutput": {"hookEventName": "SessionStart", "additionalContext": "Your injected context here"}}
```

**Cursor/Other:**
```json
{"additional_context": "Your injected context here"}
```

The `session-start` hook reads the `using-devtools` skill content and injects it into every session, similar to how superpowers injects `using-superpowers`.

---

## 7. MCP Server Configuration

### `.mcp.json` (template — copy to project)

> **Note:** Claude Code reads `.mcp.json` from project roots and `~/.claude/.mcp.json`, not from plugin directories. This file is a **template** — copy it into your project or home `.claude/` directory to use.

```json
{
  "mcpServers": {}
}
```

Add server configs as needed. Example entry:
```json
{
  "mcpServers": {
    "my-server": {
      "command": "npx",
      "args": ["my-mcp-server"],
      "env": {}
    }
  }
}
```

---

## 8. Cross-Platform Tool Mapping

### `skills/using-devtools/references/codex-tools.md`

| Claude Code | Codex Equivalent |
|------------|-----------------|
| Read       | ReadFile        |
| Write      | WriteFile       |
| Edit       | PatchFile       |
| Glob       | Glob            |
| Grep       | Grep            |
| Bash       | Shell           |

---

## 9. Workflow: Adding New Components

### Adding a skill
1. `cp -r skills/_template skills/my-skill`
2. Edit `skills/my-skill/SKILL.md` — update frontmatter and content
3. Commit and push
4. `/plugin update devtools` on each machine

### Adding an agent
1. `cp agents/_template.md agents/my-agent.md`
2. Edit frontmatter and content
3. Commit and push

### Adding a command
1. `cp commands/_template.md commands/my-command.md`
2. Edit frontmatter and content
3. Commit and push
4. Available as `/my-command` in Claude Code

### Adding a hook
1. Add event entry to `hooks/hooks.json`
2. Create corresponding script in `hooks/`
3. Commit and push

### Adding an MCP server
1. Add server config to `.mcp.json` (updates the template in the repo)
2. Commit and push
3. Copy the updated `.mcp.json` into your project root or `~/.claude/` on each machine

---

## 10. Auto-Capture Memory System

### Overview

A hook-driven + command system that detects moments worth remembering during sessions, asks for confirmation before saving, and stores memories both locally and in the plugin repo for cross-machine sync.

### What Gets Auto-Captured

| Category | Trigger Examples | Memory Type |
|----------|-----------------|-------------|
| User corrections/feedback | "No, do it this way", "don't use X" | `feedback` |
| Project discoveries | AI learns architecture patterns, file conventions, quirks | `project` |
| Tool/workflow preferences | User consistently chooses pnpm, prefers functional style | `user` |
| Debugging solutions | Tricky bug solved — root cause + fix pattern | `reference` |

### Behavior: Confirm Before Save

The system does NOT save silently. When it detects something worth remembering:

```
> I noticed you prefer using `pnpm` over `npm`. Want me to remember this
> for future sessions? (y/n)
```

Only saves if the user confirms. Never auto-saves without consent.

### Detection Mechanism

**Why a hook, not a passive skill:** Skills are triggered by user intent (e.g., "brainstorm X"). Auto-memory needs to run passively throughout a session. This is achieved via two mechanisms:

1. **SessionStart hook injection:** The `session-start` hook injects instructions telling the model to watch for memory-worthy patterns (corrections, preferences, discoveries, debugging resolutions). This makes the model aware throughout the session without requiring a skill invocation.

2. **Manual command fallback:** `/remember <description>` — user explicitly tells the model to save something. Works when auto-detection misses a moment.

### Storage: Dual-Location

**Primary (local):** Claude's built-in memory system at `~/.claude/projects/<project-hash>/memory/`
- Uses Claude Code's native memory file format (markdown with YAML frontmatter)
- Writes memory files and updates `MEMORY.md` index
- Per-project for `project` type memories
- User-level at `~/.claude/memory/` for `user` and `feedback` type memories (global preferences)
- Immediate — no commit/push needed

**Secondary (portable):** Plugin repo at `devtools/memories/`
- Travels with the plugin across machines
- Requires commit + push to sync
- Accessed via `/memory-export` command

### Components

#### Hook Injection (via `session-start`)

The `session-start` hook injects these instructions into every session:

```
Watch for these patterns during our conversation:
- If I correct your approach, offer to save as a feedback memory
- If you discover a project convention, offer to save as a project memory
- If I state a tool/style preference, offer to save as a user memory
- If we solve a tricky bug, offer to save the root cause as a reference memory

Always ask before saving. Format: "Want me to remember this for future sessions?"
```

#### Command: `commands/remember.md`

Manual memory capture — user explicitly saves something:
- Usage: `/remember prefer pnpm over npm for all projects`
- Prompts for memory type (user/feedback/project/reference) if not obvious
- Saves to appropriate local path based on type

#### Command: `commands/memory-export.md`

Exports selected local memories into `devtools/memories/` for cross-machine portability:
1. Scans both `~/.claude/memory/` (user-level) and `~/.claude/projects/<current>/memory/` (project-level)
2. Lists found memories conversationally
3. User tells the model which to export (e.g., "all", "just the pnpm one", "all feedback type")
4. Copies to `devtools/memories/<type>/` with proper frontmatter
5. Prompts user to commit and push

#### Command: `commands/memory-import.md`

Imports portable memories from plugin repo into local Claude memory on a new machine:
1. Reads `devtools/memories/**/*.md`
2. Routes by `type` frontmatter field:
   - `user` and `feedback` → `~/.claude/memory/` (global)
   - `project` → `~/.claude/projects/<current>/memory/` (project-specific)
   - `reference` → `~/.claude/memory/` (global)
3. Updates `MEMORY.md` index at each target location

#### Command: `commands/memory-cleanup.md`

Reviews accumulated memories for staleness:
1. Lists all memories with `captured` date and `type`
2. Flags entries older than a configurable threshold (default: 90 days)
3. User confirms which to keep/remove conversationally
4. Removes confirmed entries from both local and portable storage

### Repo Structure Addition

```
devtools/
├── memories/                    # Portable memories (cross-machine sync)
│   ├── MEMORY.md               # Index of portable memories
│   ├── user/                   # User preferences and profile
│   ├── feedback/               # Corrections and behavioral guidance
│   ├── project/                # Cross-project discoveries
│   └── reference/              # Debugging solutions, external pointers
├── commands/
│   ├── remember.md             # Manual memory capture
│   ├── memory-export.md        # Export local → plugin repo
│   ├── memory-import.md        # Import plugin repo → local
│   └── memory-cleanup.md       # Review and prune stale memories
```

Note: Auto-detection lives in the `session-start` hook injection, not as a separate skill.

### Memory File Format (Portable)

```markdown
---
name: prefer-pnpm
description: User prefers pnpm over npm for package management
type: user
captured: 2026-03-14
source-project: my-web-app
last-reviewed: 2026-03-14
---

Always use `pnpm` instead of `npm` for package management.

**Why:** User finds pnpm faster and prefers its strict dependency handling.
**How to apply:** When running install/add/remove commands, use pnpm. When creating new projects, initialize with pnpm.
```

### Sync Workflow

```
Machine A (working):
  Session hook detects preference → confirms with user → saves locally
  User runs /memory-export → selects memories conversationally → copies to devtools/memories/
  User commits + pushes

Machine B (new session):
  /plugin update devtools → pulls latest memories
  User runs /memory-import → routes to correct local paths by type
  Future sessions on Machine B now have the memories
```

---

## 11. Project Conventions (CLAUDE.md)

- Skills go in `skills/<name>/SKILL.md` with YAML frontmatter
- Agents go in `agents/<name>.md` with YAML frontmatter
- Commands go in `commands/<name>.md` with YAML frontmatter
- Copy `_template` in each directory as starting point
- Keep skills focused — one clear purpose per skill
- Descriptions must be specific — they control when skills trigger
- Test on Claude Code before pushing (primary platform)
- Version bump `plugin.json` on meaningful changes

---

## 12. Future Extensibility

| When | Action |
|------|--------|
| Adopt Cursor | Fill `.cursor-plugin/plugin.json` |
| Adopt Gemini CLI | Fill `gemini-extension.json` + `GEMINI.md` + `references/gemini-tools.md` |
| Adopt OpenCode | Fill `.opencode/INSTALL.md` |
| Need multiple plugins | Split into marketplace + separate plugin repos (Approach B) |
| Need team sharing | Add `.claude/settings.json` with `extraKnownMarketplaces` to project repos |
