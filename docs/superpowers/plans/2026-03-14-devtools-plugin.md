# DevTools Plugin Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create a personal portable Claude Code plugin repo with skills, agents, commands, hooks, memory system, and cross-platform templates -- installable with one command on any machine.

**Architecture:** Single GitHub repo that serves as both a Claude Code plugin and a self-hosted marketplace. Components are discovered by convention. A SessionStart hook injects auto-memory detection and meta-skill context. Memory commands enable cross-machine sync via the plugin repo.

**Tech Stack:** Markdown (YAML frontmatter), JSON (plugin configs), Bash (hooks), CMD (Windows polyglot wrapper)

**Spec:** `docs/superpowers/specs/2026-03-14-devtools-plugin-design.md`

---

## File Map

| File | Responsibility |
|------|---------------|
| `.claude-plugin/plugin.json` | Plugin identity and metadata |
| `.claude-plugin/marketplace.json` | Self-hosted marketplace manifest |
| `.codex/INSTALL.md` | Codex installation guide |
| `.gitignore` | Ignore patterns |
| `.gitattributes` | Line ending normalization |
| `skills/_template/SKILL.md` | Starter template for new skills |
| `skills/using-devtools/SKILL.md` | Meta-skill: platform mapping and usage guide |
| `skills/using-devtools/references/codex-tools.md` | Claude Code -> Codex tool name mapping |
| `agents/_template.md` | Starter template for new agents |
| `commands/_template.md` | Starter template for new commands |
| `commands/remember.md` | Manual memory capture command |
| `commands/memory-export.md` | Export local memories -> plugin repo |
| `commands/memory-import.md` | Import plugin repo memories -> local |
| `commands/memory-cleanup.md` | Review and prune stale memories |
| `hooks/hooks.json` | Hook event registrations |
| `hooks/session-start` | Bash SessionStart hook script |
| `hooks/run-hook.cmd` | Windows polyglot wrapper |
| `.mcp.json` | MCP server config template |
| `memories/MEMORY.md` | Portable memory index |
| `README.md` | Usage and installation docs |
| `CLAUDE.md` | Project conventions |

---

## Chunk 1: Core Plugin Scaffold

### Task 1: Initialize Git Repo and Config Files

**Files:**
- Create: `.gitignore`
- Create: `.gitattributes`

- [ ] **Step 1: Initialize git repo**

Run: `cd C:/Users/91807/Documents/Portfolio/DEVTOOLS && git init`
Expected: `Initialized empty Git repository`

- [ ] **Step 2: Create .gitignore**

Create `.gitignore`:
```
# OS
.DS_Store
Thumbs.db
Desktop.ini

# Editors
*.swp
*.swo
*~
.vscode/
.idea/

# Node (for MCP servers)
node_modules/

# Env files
.env
.env.*
```

- [ ] **Step 3: Create .gitattributes**

Create `.gitattributes`:
```
# Normalize line endings
* text=auto

# Bash scripts must use LF
hooks/session-start text eol=lf
hooks/run-hook.cmd text eol=crlf

# Markdown
*.md text eol=lf
```

- [ ] **Step 4: Commit**

```bash
git add .gitignore .gitattributes
git commit -m "chore: initialize repo with gitignore and gitattributes"
```

---

### Task 2: Plugin and Marketplace Configuration

**Files:**
- Create: `.claude-plugin/plugin.json`
- Create: `.claude-plugin/marketplace.json`

- [ ] **Step 1: Create .claude-plugin directory**

Run: `mkdir -p .claude-plugin`

- [ ] **Step 2: Create plugin.json**

Create `.claude-plugin/plugin.json`:
```json
{
  "name": "devtools",
  "description": "Personal portable toolkit -- skills, agents, commands, and hooks across all projects",
  "version": "1.0.0",
  "author": { "name": "your-name" },
  "homepage": "https://github.com/your-username/devtools",
  "repository": "https://github.com/your-username/devtools",
  "license": "MIT",
  "keywords": ["toolkit", "skills", "agents", "portable", "personal"]
}
```

- [ ] **Step 3: Create marketplace.json**

Create `.claude-plugin/marketplace.json`:
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

- [ ] **Step 4: Commit**

```bash
git add .claude-plugin/
git commit -m "feat: add plugin and marketplace configuration"
```

---

### Task 3: Codex Platform Support

**Files:**
- Create: `.codex/INSTALL.md`

- [ ] **Step 1: Create .codex directory**

Run: `mkdir -p .codex`

- [ ] **Step 2: Create INSTALL.md**

Create `.codex/INSTALL.md`:
```markdown
# DevTools -- Codex Installation

## Setup

1. Clone the repo:

   ```bash
   git clone https://github.com/your-username/devtools.git ~/.codex/devtools
   ```

2. Symlink skills into Codex's skills directory:

   ```bash
   mkdir -p ~/.agents/skills
   ln -s ~/.codex/devtools/skills ~/.agents/skills/devtools
   ```

   > **Note:** Verify symlink path against current Codex docs -- path may vary by version.

3. For tool name differences between Claude Code and Codex, see:
   `skills/using-devtools/references/codex-tools.md`

## Updating

```bash
cd ~/.codex/devtools && git pull
```
```

- [ ] **Step 3: Commit**

```bash
git add .codex/
git commit -m "feat: add Codex installation guide"
```

---

### Task 4: Component Templates

**Files:**
- Create: `skills/_template/SKILL.md`
- Create: `agents/_template.md`
- Create: `commands/_template.md`

- [ ] **Step 1: Create directory structure**

Run: `mkdir -p skills/_template agents commands`

- [ ] **Step 2: Create skill template**

Create `skills/_template/SKILL.md`:
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

- [ ] **Step 3: Create agent template**

Create `agents/_template.md`:
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

- [ ] **Step 4: Create command template**

Create `commands/_template.md`:
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

- [ ] **Step 5: Commit**

```bash
git add skills/_template/ agents/_template.md commands/_template.md
git commit -m "feat: add skill, agent, and command templates"
```

---

### Task 5: Using-DevTools Meta-Skill

**Files:**
- Create: `skills/using-devtools/SKILL.md`
- Create: `skills/using-devtools/references/codex-tools.md`

- [ ] **Step 1: Create directory structure**

Run: `mkdir -p skills/using-devtools/references`

- [ ] **Step 2: Create using-devtools skill**

Create `skills/using-devtools/SKILL.md`:
```markdown
---
name: using-devtools
description: Meta-skill -- guides tool name mapping when running on non-Claude-Code platforms and provides overview of available devtools components
---

# Using DevTools

This is your personal portable toolkit. It contains skills, agents, commands, hooks, and memory management -- installed as a single Claude Code plugin.

## Available Components

Check these directories for what's available:
- `skills/` -- workflow skills (auto-triggered based on description match)
- `agents/` -- specialized subagents
- `commands/` -- slash commands (e.g., `/remember`, `/memory-export`)
- `hooks/` -- lifecycle event triggers
- `memories/` -- portable cross-machine memories

## Platform Support

This plugin uses Claude Code tool names by default. When running on other platforms, check the `references/` directory for tool name mappings:
- Codex: `references/codex-tools.md`

## Memory System

This plugin includes an auto-capture memory system:
- Corrections, preferences, discoveries, and debugging solutions are detected during sessions
- You will be asked before anything is saved
- Use `/remember` to manually save something
- Use `/memory-export` to sync memories across machines
- Use `/memory-import` to load memories on a new machine
- Use `/memory-cleanup` to review and prune stale memories
```

- [ ] **Step 3: Create codex tools mapping**

Create `skills/using-devtools/references/codex-tools.md`:
```markdown
# Codex Tool Mapping

When running on Codex, use these equivalent tool names:

| Claude Code | Codex Equivalent |
|------------|-----------------|
| Read       | ReadFile        |
| Write      | WriteFile       |
| Edit       | PatchFile       |
| Glob       | Glob            |
| Grep       | Grep            |
| Bash       | Shell           |
```

- [ ] **Step 4: Commit**

```bash
git add skills/using-devtools/
git commit -m "feat: add using-devtools meta-skill with Codex tool mapping"
```

---

### Task 6: Hooks -- SessionStart with Auto-Memory Injection

**Files:**
- Create: `hooks/hooks.json`
- Create: `hooks/session-start`
- Create: `hooks/run-hook.cmd`

- [ ] **Step 1: Create hooks directory**

Run: `mkdir -p hooks`

- [ ] **Step 2: Create hooks.json**

Create `hooks/hooks.json`:
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

- [ ] **Step 3: Create session-start hook**

Create `hooks/session-start`:
```bash
#!/usr/bin/env bash
# SessionStart hook for devtools plugin
# Injects using-devtools skill content + auto-memory detection instructions

set -euo pipefail

# Determine plugin root directory
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]:-$0}")" && pwd)"
PLUGIN_ROOT="$(cd "${SCRIPT_DIR}/.." && pwd)"

# Read using-devtools content
using_devtools_content=$(cat "${PLUGIN_ROOT}/skills/using-devtools/SKILL.md" 2>&1 || echo "Error reading using-devtools skill")

# Escape string for JSON embedding using bash parameter substitution
escape_for_json() {
    local s="$1"
    s="${s//\\/\\\\}"
    s="${s//\"/\\\"}"
    s="${s//$'\n'/\\n}"
    s="${s//$'\r'/\\r}"
    s="${s//$'\t'/\\t}"
    printf '%s' "$s"
}

using_devtools_escaped=$(escape_for_json "$using_devtools_content")

# Auto-memory detection instructions
memory_instructions="Watch for these patterns during our conversation:\\n- If I correct your approach, offer to save as a feedback memory\\n- If you discover a project convention, offer to save as a project memory\\n- If I state a tool/style preference, offer to save as a user memory\\n- If we solve a tricky bug, offer to save the root cause as a reference memory\\n\\nAlways ask before saving. Format: \\\"Want me to remember this for future sessions?\\\""

session_context="<IMPORTANT>\\nYou have devtools installed.\\n\\n${using_devtools_escaped}\\n\\n<auto-memory>\\n${memory_instructions}\\n</auto-memory>\\n</IMPORTANT>"

# Output context injection as JSON
# Claude Code sets CLAUDE_PLUGIN_ROOT -- emit only hookSpecificOutput
# Other platforms (Cursor, etc.) -- emit only additional_context
if [ -n "${CLAUDE_PLUGIN_ROOT:-}" ]; then
  cat <<EOF
{
  "hookSpecificOutput": {
    "hookEventName": "SessionStart",
    "additionalContext": "${session_context}"
  }
}
EOF
else
  cat <<EOF
{
  "additional_context": "${session_context}"
}
EOF
fi

exit 0
```

- [ ] **Step 4: Create run-hook.cmd (Windows polyglot wrapper)**

Create `hooks/run-hook.cmd`:
```cmd
: << 'CMDBLOCK'
@echo off
REM Cross-platform polyglot wrapper for hook scripts.
REM On Windows: cmd.exe runs the batch portion, which finds and calls bash.
REM On Unix: the shell interprets this as a script (: is a no-op in bash).
REM
REM Usage: run-hook.cmd <script-name> [args...]

if "%~1"=="" (
    echo run-hook.cmd: missing script name >&2
    exit /b 1
)

set "HOOK_DIR=%~dp0"

REM Try Git for Windows bash in standard locations
if exist "C:\Program Files\Git\bin\bash.exe" (
    "C:\Program Files\Git\bin\bash.exe" "%HOOK_DIR%%~1" %2 %3 %4 %5 %6 %7 %8 %9
    exit /b %ERRORLEVEL%
)
if exist "C:\Program Files (x86)\Git\bin\bash.exe" (
    "C:\Program Files (x86)\Git\bin\bash.exe" "%HOOK_DIR%%~1" %2 %3 %4 %5 %6 %7 %8 %9
    exit /b %ERRORLEVEL%
)

REM Try bash on PATH (e.g. user-installed Git Bash, MSYS2, Cygwin)
where bash >nul 2>nul
if %ERRORLEVEL% equ 0 (
    bash "%HOOK_DIR%%~1" %2 %3 %4 %5 %6 %7 %8 %9
    exit /b %ERRORLEVEL%
)

REM No bash found - exit silently rather than error
exit /b 0
CMDBLOCK

# Unix: run the named script directly
SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
SCRIPT_NAME="$1"
shift
exec bash "${SCRIPT_DIR}/${SCRIPT_NAME}" "$@"
```

- [ ] **Step 5: Make hook scripts executable**

Run: `chmod +x hooks/session-start hooks/run-hook.cmd`

- [ ] **Step 6: Commit**

```bash
git add hooks/
git commit -m "feat: add SessionStart hook with auto-memory detection injection"
```

---

### Task 7: MCP Server Template

**Files:**
- Create: `.mcp.json`

- [ ] **Step 1: Create .mcp.json**

Create `.mcp.json`:
```json
{
  "mcpServers": {}
}
```

- [ ] **Step 2: Commit**

```bash
git add .mcp.json
git commit -m "feat: add MCP server config template"
```

---

## Chunk 2: Memory System

### Task 8: Portable Memory Directory Structure

**Files:**
- Create: `memories/MEMORY.md`
- Create: `memories/user/.gitkeep`
- Create: `memories/feedback/.gitkeep`
- Create: `memories/project/.gitkeep`
- Create: `memories/reference/.gitkeep`

- [ ] **Step 1: Create memory directories**

Run: `mkdir -p memories/user memories/feedback memories/project memories/reference`

- [ ] **Step 2: Create MEMORY.md index**

Create `memories/MEMORY.md`:
```markdown
# Portable Memories

Index of memories synced across machines via this plugin repo.

## User Preferences
<!-- Add links to user/ memories here -->

## Feedback & Corrections
<!-- Add links to feedback/ memories here -->

## Project Discoveries
<!-- Add links to project/ memories here -->

## References
<!-- Add links to reference/ memories here -->
```

- [ ] **Step 3: Create .gitkeep files to preserve empty directories**

Run: `touch memories/user/.gitkeep memories/feedback/.gitkeep memories/project/.gitkeep memories/reference/.gitkeep`

- [ ] **Step 4: Commit**

```bash
git add memories/
git commit -m "feat: add portable memory directory structure"
```

---

### Task 9: /remember Command

**Files:**
- Create: `commands/remember.md`

- [ ] **Step 1: Create remember.md**

Create `commands/remember.md`:
```markdown
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
```

- [ ] **Step 2: Commit**

```bash
git add commands/remember.md
git commit -m "feat: add /remember command for manual memory capture"
```

---

### Task 10: /memory-export Command

**Files:**
- Create: `commands/memory-export.md`

- [ ] **Step 1: Create memory-export.md**

Create `commands/memory-export.md`:
```markdown
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
```

- [ ] **Step 2: Commit**

```bash
git add commands/memory-export.md
git commit -m "feat: add /memory-export command for cross-machine sync"
```

---

### Task 11: /memory-import Command

**Files:**
- Create: `commands/memory-import.md`

- [ ] **Step 1: Create memory-import.md**

Create `commands/memory-import.md`:
```markdown
---
description: Import portable memories from plugin repo into local Claude memory
argument-hint: [all | <memory-name> | <type>]
allowed-tools: [Read, Write, Glob, Grep, Bash]
---

# Memory Import

$ARGUMENTS

## Instructions

Import memories from the devtools plugin repo into local Claude memory on this machine.

1. **Find the plugin repo path** -- locate the devtools plugin installation:
   - Check `~/.claude/plugins/installed_plugins.json` for the devtools install path
   - The `memories/` directory is at `<install-path>/memories/`

2. **Scan portable memories:**
   - Read all `.md` files in `<plugin-repo>/memories/user/`, `feedback/`, `project/`, `reference/`
   - Skip `.gitkeep` files and `MEMORY.md`

3. **Filter based on $ARGUMENTS:**
   - `all` -- import everything
   - A specific name -- import only that memory
   - A type -- import all of that type
   - No argument -- list all available and ask the user which to import

4. **Route each memory by type** (read from frontmatter):
   - `user` or `feedback` -> write to `~/.claude/memory/` (global)
   - `project` -> write to `~/.claude/projects/<current-project-hash>/memory/` (project-specific)
   - `reference` -> write to `~/.claude/memory/` (global)

5. **Check for duplicates** -- if a memory with the same `name` already exists at the target location, ask the user whether to overwrite or skip.

6. **Update MEMORY.md** index at each target location.

7. **Confirm** -- tell the user how many memories were imported and where.
```

- [ ] **Step 2: Commit**

```bash
git add commands/memory-import.md
git commit -m "feat: add /memory-import command for loading memories on new machines"
```

---

### Task 12: /memory-cleanup Command

**Files:**
- Create: `commands/memory-cleanup.md`

- [ ] **Step 1: Create memory-cleanup.md**

Create `commands/memory-cleanup.md`:
```markdown
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
```

- [ ] **Step 2: Commit**

```bash
git add commands/memory-cleanup.md
git commit -m "feat: add /memory-cleanup command for pruning stale memories"
```

---

## Chunk 3: Platform Placeholders and Missing Files

### Task 13: Cursor Plugin Config (Future -- Full Content from Spec)

**Files:**
- Create: `.cursor-plugin/plugin.json`

- [ ] **Step 1: Create .cursor-plugin directory**

Run: `mkdir -p .cursor-plugin`

- [ ] **Step 2: Create plugin.json with explicit component pointers**

Create `.cursor-plugin/plugin.json`:
```json
{
  "name": "devtools",
  "displayName": "DevTools",
  "description": "Personal portable toolkit -- skills, agents, commands, and hooks across all projects",
  "version": "1.0.0",
  "author": { "name": "your-name" },
  "license": "MIT",
  "skills": "./skills/",
  "agents": "./agents/",
  "commands": "./commands/",
  "hooks": "./hooks/hooks.json"
}
```

- [ ] **Step 3: Commit**

```bash
git add .cursor-plugin/
git commit -m "feat: add Cursor plugin config with component pointers"
```

---

### Task 14: Gemini CLI and OpenCode Placeholders

**Files:**
- Create: `gemini-extension.json`
- Create: `GEMINI.md`
- Create: `skills/using-devtools/references/gemini-tools.md`
- Create: `.opencode/INSTALL.md`

- [ ] **Step 1: Create gemini-extension.json**

Create `gemini-extension.json`:
```json
{
  "name": "devtools",
  "description": "Personal portable toolkit -- skills, agents, commands, and hooks across all projects",
  "version": "1.0.0",
  "contextFileName": "GEMINI.md"
}
```

- [ ] **Step 2: Create GEMINI.md placeholder**

Create `GEMINI.md`:
```markdown
# DevTools -- Gemini CLI

> This file is a placeholder. Fill in when adopting Gemini CLI.

See `skills/using-devtools/references/gemini-tools.md` for tool name mapping.
```

- [ ] **Step 3: Create gemini-tools.md**

Create `skills/using-devtools/references/gemini-tools.md`:
```markdown
# Gemini CLI Tool Mapping

> Placeholder -- fill in when adopting Gemini CLI.

| Claude Code | Gemini Equivalent |
|------------|------------------|
| Read       | TBD              |
| Write      | TBD              |
| Edit       | TBD              |
| Glob       | TBD              |
| Grep       | TBD              |
| Bash       | TBD              |
```

- [ ] **Step 4: Create .opencode/INSTALL.md**

Run: `mkdir -p .opencode`

Create `.opencode/INSTALL.md`:
```markdown
# DevTools -- OpenCode Installation

> This file is a placeholder. Fill in when adopting OpenCode.
```

- [ ] **Step 5: Commit**

```bash
git add gemini-extension.json GEMINI.md skills/using-devtools/references/gemini-tools.md .opencode/
git commit -m "feat: add Gemini CLI and OpenCode platform placeholders"
```

---

### Task 15: Auto-Memory Skill Directory

The auto-memory detection lives in the session-start hook injection (Section 10 of spec). However, the spec's repo structure lists `skills/auto-memory/SKILL.md` as a directory entry. This skill serves as documentation and a manual trigger for users who want to explicitly invoke memory detection mid-session.

**Files:**
- Create: `skills/auto-memory/SKILL.md`

- [ ] **Step 1: Create auto-memory skill directory**

Run: `mkdir -p skills/auto-memory`

- [ ] **Step 2: Create SKILL.md**

Create `skills/auto-memory/SKILL.md`:
```markdown
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
```

- [ ] **Step 3: Commit**

```bash
git add skills/auto-memory/
git commit -m "feat: add auto-memory skill for memory-worthy moment detection"
```

---

## Chunk 4: Documentation and Final Assembly

### Task 16: README.md

**Files:**
- Create: `README.md`

- [ ] **Step 1: Create README.md**

Create `README.md`:
```markdown
# DevTools

Personal portable toolkit for AI-assisted development.
Skills, agents, commands, and hooks -- one install, every machine.

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
```

- [ ] **Step 2: Commit**

```bash
git add README.md
git commit -m "docs: add README with installation and usage guide"
```

---

### Task 17: CLAUDE.md

**Files:**
- Create: `CLAUDE.md`

- [ ] **Step 1: Create CLAUDE.md**

Create `CLAUDE.md`:
```markdown
# DevTools -- Project Conventions

## What This Repo Is

Personal portable Claude Code plugin. Contains skills, agents, commands, hooks, and memory management.

## Directory Layout

- `skills/<name>/SKILL.md` -- workflow skills with YAML frontmatter
- `agents/<name>.md` -- subagent definitions with YAML frontmatter
- `commands/<name>.md` -- slash commands with YAML frontmatter
- `hooks/` -- lifecycle event hooks (bash scripts + JSON config)
- `memories/` -- portable cross-machine memories
- `.claude-plugin/` -- plugin and marketplace metadata
- `.codex/` -- Codex platform support

## Adding Components

Copy the `_template` file in each directory as your starting point.

## Rules

- Keep skills focused -- one clear purpose per skill
- Descriptions must be specific -- they control when skills trigger
- Test on Claude Code before pushing (primary platform)
- Version bump `.claude-plugin/plugin.json` on meaningful changes
- Never commit secrets or `.env` files
- Hook scripts must output valid JSON (see hooks/session-start for format)
```

- [ ] **Step 2: Commit**

```bash
git add CLAUDE.md
git commit -m "docs: add CLAUDE.md with project conventions"
```

---

### Task 18: Create Tests Placeholder

**Files:**
- Create: `tests/.gitkeep`

- [ ] **Step 1: Create tests directory**

```bash
mkdir -p tests
touch tests/.gitkeep
```

- [ ] **Step 2: Commit**

```bash
git add tests/
git commit -m "chore: add tests placeholder directory"
```

---

### Task 19: Verify Plugin Structure

- [ ] **Step 1: Verify all expected files exist**

Run:
```bash
echo "=== Checking required files ===" && \
for f in \
  .claude-plugin/plugin.json \
  .claude-plugin/marketplace.json \
  .codex/INSTALL.md \
  .cursor-plugin/plugin.json \
  .opencode/INSTALL.md \
  .gitignore \
  .gitattributes \
  gemini-extension.json \
  GEMINI.md \
  skills/_template/SKILL.md \
  skills/using-devtools/SKILL.md \
  skills/using-devtools/references/codex-tools.md \
  skills/using-devtools/references/gemini-tools.md \
  skills/auto-memory/SKILL.md \
  agents/_template.md \
  commands/_template.md \
  commands/remember.md \
  commands/memory-export.md \
  commands/memory-import.md \
  commands/memory-cleanup.md \
  hooks/hooks.json \
  hooks/session-start \
  hooks/run-hook.cmd \
  .mcp.json \
  memories/MEMORY.md \
  README.md \
  CLAUDE.md; do
  if [ -f "$f" ]; then echo "OK: $f"; else echo "MISSING: $f"; fi
done
```

Expected: All files show `OK`

- [ ] **Step 2: Verify JSON files are valid**

Run:
```bash
python -m json.tool .claude-plugin/plugin.json > /dev/null && echo "plugin.json: valid" && \
python -m json.tool .claude-plugin/marketplace.json > /dev/null && echo "marketplace.json: valid" && \
python -m json.tool hooks/hooks.json > /dev/null && echo "hooks.json: valid" && \
python -m json.tool .mcp.json > /dev/null && echo ".mcp.json: valid"
```

Expected: All files show `valid`

- [ ] **Step 3: Verify session-start hook runs**

Run:
```bash
CLAUDE_PLUGIN_ROOT="$(pwd)" bash hooks/session-start | python -m json.tool
```

Expected: Valid JSON output with `hookSpecificOutput.additionalContext` containing the using-devtools skill content and auto-memory instructions

- [ ] **Step 4: Verify git log**

Run: `git log --oneline`

Expected: Clean commit history with all tasks committed
