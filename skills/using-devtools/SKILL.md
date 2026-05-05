---
name: using-devtools
description: Meta-skill -- guides tool name mapping when running on non-Claude-Code platforms and provides overview of available devtools components
---

# Using DevTools

This is your personal portable toolkit. It contains skills, agents, commands, hooks, and memory management -- distributed as a single plugin/extension and supported across multiple AI development platforms (Claude Code, Codex, Gemini CLI).

## Available Components

Check these directories for what's available:
- `skills/` -- workflow skills (auto-triggered based on description match)
- `agents/` -- specialized subagents
- `commands/` -- slash commands (e.g., `/remember`, `/memory-export`)
- `hooks/` -- lifecycle event triggers
- `memories/` -- portable cross-machine memories

## Platform Support

This plugin uses Claude Code tool names by default. When running on other platforms, check the `references/` directory for tool name mappings:
- Claude Code: native (no mapping needed)
- Codex: `references/codex-tools.md`
- Gemini CLI: `references/gemini-tools.md`

## Memory System

This plugin includes an auto-capture memory system:
- Corrections, preferences, discoveries, and debugging solutions are detected during sessions
- You will be asked before anything is saved

Manual memory operations (slash commands — **Claude Code only** in v1; not installed on Codex or Gemini until v2a):
- Use `/remember` to manually save something
- Use `/memory-export` to sync memories across machines
- Use `/memory-import` to load memories on a new machine
- Use `/memory-cleanup` to review and prune stale memories

On Codex and Gemini, persistence is platform-native: Gemini exposes a `save_memory` tool; Codex relies on shell-level file writes into `~/.agents/memory/` (or equivalent). The auto-detection conversation pattern is identical across platforms.
