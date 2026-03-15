---
description: Use an agent standalone or dispatch multiple agents in parallel
argument-hint: <agent-name> <task> | <agent1>,<agent2> <task> | --list | --generic <agent-name> <task>
allowed-tools: [Read, Write, Edit, Glob, Grep, Bash, Agent]
---

# Agent

$ARGUMENTS

## Instructions

Dispatch one or more agents directly without full orchestration.

### Parse Arguments

- `--list`: show available agents and exit
- `--generic <name> <task>`: use base template without project-specific context
- `<name> <task>`: dispatch a single project-specific agent
- `<name1>,<name2> <task>`: dispatch multiple agents in parallel

### --list Mode

1. Check `.claude/agents/` for project-specific agents
2. Find devtools plugin path from `~/.claude/plugins/installed_plugins.json`
3. List base agents from `<plugin-path>/agents/_bases/`
4. Display:
   ```
   Project agents (.claude/agents/):
     react-frontend-developer  — React frontend specialist
     node-backend-developer    — Node.js backend specialist
     fullstack-tester          — Cross-service test writer

   Base agents (available with --generic):
     requirements, researcher, architect, ux-designer, developer,
     tester, reviewer, security, devops, docs-writer, cost-optimizer,
     cloud-aws, cloud-gcp, cloud-azure, cloud-terraform, cloud-pulumi
   ```

### Single Agent Mode

1. Look for `.claude/agents/<name>.md` (project-specific)
2. If not found:
   - If `--generic` flag: read base template from devtools plugin, strip `{{PLACEHOLDER}}` lines, add generic instruction "Analyze the current project context before proceeding."
   - If no flag: tell user "No project agent found. Run `/assemble-team` first, or use `--generic` for a basic agent."
3. Dispatch as subagent with the Agent tool, passing the task as the prompt

### Multiple Agent Mode

1. Split agent names by comma
2. Check all agents exist (project or generic)
3. Verify parallel safety:
   - Read-only agents (researcher, reviewer, security, architect, requirements, ux-designer, cost-optimizer): always safe in parallel
   - Write agents (developer, tester, devops, docs-writer): safe only if scoped to different directories
   - If unsafe: run sequentially instead, tell user why
4. Dispatch all agents in parallel using multiple Agent tool calls in a single message
5. Collect and present results

### Output

- Small results: display directly in conversation
- Large results: agent writes to file, display file path
