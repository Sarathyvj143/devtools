---
description: Run a phased multi-agent workflow on a task
argument-hint: <task description> [--phases N,N] [--resume]
allowed-tools: [Read, Write, Edit, Glob, Grep, Bash, Agent]
---

# Orchestrate

$ARGUMENTS

## Instructions

Run a phased multi-agent workflow on the given task. This command invokes the orchestrator skill.

### Prerequisites Check

1. Verify `.claude/agents/` exists and contains agent files
2. Verify `.claude/team-config.json` exists
3. If missing: tell user to run `/assemble-team` first and stop

### Parse Arguments

- Task description: everything that is not a flag
- `--phases N,N,N`: skip to specific phases (e.g., `--phases 4,5,6`)
- `--resume`: resume the most recent interrupted run

### Invoke Orchestrator

Use the `Skill` tool to invoke `devtools:orchestrator` with the parsed task and flags.

The orchestrator skill handles:
- Workflow pattern selection (simple/medium/complex)
- Existing work detection (specs, plans, previous runs)
- Phase execution with agent dispatch
- Cross-verification gates
- Run log management
- Resume support

### Run Directory Setup

If not resuming, create:
```
.claude/orchestrator/runs/YYYY-MM-DD-<task-slug>/
  run-log.md
```

The task slug is derived from the task description (lowercase, hyphens, max 50 chars).
