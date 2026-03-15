# Multi-Agent Orchestration System — Design Spec

**Date:** 2026-03-15
**Status:** Approved
**Approach:** Template-Based Generation + Phased Orchestration

---

## 1. Overview

A multi-agent orchestration system for the devtools plugin that assembles project-specific agent teams, runs them in configurable workflow patterns (parallel, sequential, hub-spoke), and includes cross-verification gates between phases. Agents are generated from base templates tailored to the detected project type and services.

### Goals
- 11 specialized agent roles covering the full development lifecycle
- Auto-detect project type and services (including multi-service projects)
- Generate project-specific agents from base templates + service profiles
- Orchestrate agents in phased workflows with review gates and cross-verification
- Agents usable both orchestrated (via `/orchestrate`) and standalone (via `/agent`)
- Integrate with existing superpowers flow (brainstorming → writing-plans → executing-plans → finishing)
- Audit existing agents with 80% quality threshold — regenerate stale agents

### Non-Goals
- Not a replacement for superpowers' built-in skills — integrates with them
- Not a CI/CD system — agents work within Claude Code sessions
- No persistent agent state between sessions beyond generated files

---

## 2. Architecture: Controller + Agents Model

### How Orchestration Works in Claude Code

Claude Code uses a **flat controller-to-subagent** model:
- The **controller** is the top-level session (the main Claude Code instance)
- **Subagents** are dispatched by the controller — they cannot dispatch further subagents
- Subagents have isolated context and return results to the controller

**The "Manager" is NOT an agent.** It is a **skill** (`skills/orchestrator/SKILL.md`) that instructs the top-level controller how to run phases, dispatch agents, and make decisions. The orchestration logic lives in this skill, not in a dispatched subagent.

### Parallel Write Safety

Multiple write-agents editing the same repository will conflict. Parallel dispatch is only safe when:
- **Read-only agents** (researcher, reviewer, security) — always safe to parallelize
- **Write-agents with file-scope isolation** — each developer is scoped to a specific service directory (e.g., Developer A only touches `./frontend/`, Developer B only touches `./backend/`)
- **Sequential fallback** — if agents share file scope, they run sequentially

### Context Window Management

Each phase dispatches fresh subagents (resetting context). The controller maintains only summary state between phases via the `run-log.md` file. This prevents context overflow during complex multi-phase workflows.

---

## 3. Agent Roster (10 Subagents + 1 Skill)

| # | Type | Name | Role | Phase | Tools |
|---|------|------|------|-------|-------|
| — | Skill | Orchestrator | Phase management, workflow decisions, agent dispatch | All | (controller-level) |
| 1 | Agent | Requirements Analyst | Gathers and clarifies requirements, writes specs | 1 | Read, Glob, Grep, Bash |
| 2 | Agent | Researcher | Investigates tech, finds solutions, reads docs | 1 | Read, Glob, Grep, Bash, WebSearch, WebFetch |
| 3 | Agent | Architect | System design, tech stack decisions, patterns | 2 | Read, Glob, Grep, Bash |
| 4 | Agent | UX Designer | User experience, UI patterns, accessibility | 2 | Read, Glob, Grep, Bash |
| 5 | Agent | Developer | Writes implementation code | 4 | Read, Write, Edit, Glob, Grep, Bash |
| 6 | Agent | Tester | Writes and runs tests | 4 | Read, Write, Edit, Glob, Grep, Bash |
| 7 | Agent | Reviewer | Code review, quality checks | 5 | Read, Glob, Grep, Bash |
| 8 | Agent | Security Analyst | Vulnerability scanning, security review | 5 | Read, Glob, Grep, Bash |
| 9 | Agent | DevOps | CI/CD, deployment, cloud config | 4 | Read, Write, Edit, Glob, Grep, Bash |
| 10 | Agent | Documentation Writer | Generates docs, API references, guides | 6 | Read, Write, Edit, Glob, Grep, Bash |

**Note:** The Orchestrator is a skill, not an agent. It runs at the controller level and dispatches all other agents.

---

## 4. Repository Structure

```
devtools/
├── agents/
│   ├── _bases/                    # Base templates for 10 agents
│   │   # NOTE: No manager.md — orchestration is a skill, not an agent
│   │   ├── requirements.md
│   │   ├── researcher.md
│   │   ├── architect.md
│   │   ├── ux-designer.md
│   │   ├── developer.md
│   │   ├── tester.md
│   │   ├── reviewer.md
│   │   ├── security.md
│   │   ├── devops.md
│   │   └── docs-writer.md
│   ├── _profiles/
│   │   ├── services/              # Per-service context profiles
│   │   │   ├── frontend-react.json
│   │   │   ├── frontend-vue.json
│   │   │   ├── frontend-nextjs.json
│   │   │   ├── backend-node.json
│   │   │   ├── backend-python.json
│   │   │   ├── backend-go.json
│   │   │   ├── database-postgres.json
│   │   │   ├── database-mongo.json
│   │   │   ├── mobile-react-native.json
│   │   │   ├── infra-docker.json
│   │   │   ├── infra-kubernetes.json
│   │   │   └── ml-python.json
│   │   └── compositions/         # Multi-service combination rules
│   │       ├── fullstack-react-node.json
│   │       ├── fullstack-react-python.json
│   │       ├── microservices.json
│   │       └── general.json      # Fallback
│   └── _template.md              # Existing custom agent template
├── skills/
│   └── orchestrator/
│       └── SKILL.md              # Phase management + workflow selection
├── commands/
│   ├── assemble-team.md          # Detect → generate → audit
│   ├── orchestrate.md            # Run full phased workflow
│   └── agent.md                  # Use agent(s) standalone
```

**Generated in target project:**

```
project/
├── .claude/
│   ├── agents/                   # Project-specific generated agents
│   │   ├── react-frontend-developer.md
│   │   ├── node-backend-developer.md
│   │   ├── fullstack-tester.md
│   │   └── ...
│   ├── team-config.json          # Team state, scores, timestamps
│   └── orchestrator/
│       └── runs/                 # Per-task run outputs
│           └── YYYY-MM-DD-<task-name>/
│               ├── requirements.md
│               ├── research-report.md
│               ├── architecture.md
│               ├── ux-spec.md
│               ├── verification-report.md
│               ├── security-report.md
│               └── run-log.md
```

---

## 5. Base Agent Template Format

Each base agent in `agents/_bases/` uses placeholders replaced during generation:

```markdown
---
name: developer
description: Writes implementation code following project patterns
model: inherit
allowed-tools: [Read, Write, Edit, Glob, Grep, Bash]
---

# Developer Agent

You are a senior developer working on {{PROJECT_NAME}}.

## Tech Stack
{{TECH_STACK}}

## Project Structure
{{PROJECT_STRUCTURE}}

## Project Conventions
{{CONVENTIONS}}

## Your Assigned Service
{{SERVICE_NAME}} — {{SERVICE_PATH}}

## Your Task
- Write clean, tested, production-ready code
- Follow existing patterns in the codebase
- Keep files focused — one responsibility per file

## Rules
- Never modify files outside your assigned scope
- Run existing tests before and after changes
- Commit after each logical unit of work

## Output
Write results to: {{OUTPUT_DIR}}/developer-output.md
```

Placeholder resolution:
- `{{PROJECT_NAME}}` — from git remote or directory name
- `{{TECH_STACK}}` — from detected profile context
- `{{PROJECT_STRUCTURE}}` — generated file tree of relevant directories
- `{{CONVENTIONS}}` — from profile + CLAUDE.md if exists
- `{{SERVICE_NAME}}` / `{{SERVICE_PATH}}` — from composite detection
- `{{OUTPUT_DIR}}` — `.claude/orchestrator/runs/<current-run>/`

---

## 6. Service Profile Format

### Single Service Profile (`_profiles/services/frontend-react.json`)

```json
{
  "name": "frontend-react",
  "detect": {
    "files": ["package.json"],
    "dependencies": ["react", "react-dom"],
    "patterns": ["src/**/*.tsx", "src/**/*.jsx", "**/*.css", "**/*.scss"],
    "dev_dependencies": ["vite", "webpack", "next"]
  },
  "context": {
    "framework": "React",
    "language": "TypeScript/JavaScript",
    "test_runner": "vitest or jest",
    "conventions": "Component-based architecture, hooks over classes, CSS modules or Tailwind"
  },
  "agent_customizations": {
    "developer": {
      "name_prefix": "react-frontend",
      "extra_instructions": "Use functional components with hooks. Follow React 19 patterns if detected."
    },
    "tester": {
      "extra_instructions": "Use React Testing Library for component tests. Test user interactions, not implementation details."
    },
    "ux-designer": {
      "extra_instructions": "Consider React component composition patterns. Design with props/state boundaries in mind."
    }
  },
  "phases": {
    "skip_ux": false,
    "skip_devops": true,
    "parallel_dev_test": true
  }
}
```

### Composition Profile (`_profiles/compositions/fullstack-react-node.json`)

```json
{
  "name": "fullstack-react-node",
  "requires": ["frontend-react", "backend-node"],
  "team": {
    "per_service_developers": true,
    "always": ["react-frontend-developer", "node-backend-developer", "fullstack-tester", "api-reviewer"],
    "recommended": ["architect", "devops", "security"],
    "optional": ["ux-designer", "docs-writer", "researcher", "requirements"]
  },
  "cross_service_checks": {
    "api_contract": "Verify frontend API calls match backend endpoint signatures",
    "shared_types": "Check shared TypeScript types between frontend and backend",
    "env_vars": "Verify all env vars used in frontend have backend counterparts"
  }
}
```

---

## 7. `/assemble-team` Command

### First Run — Detection and Generation

```
/assemble-team
     ↓
1. Scan project structure
   - Walk root + subdirectories (max 3 levels)
   - Check manifest files: package.json, requirements.txt, go.mod,
     Cargo.toml, pyproject.toml, pom.xml
   - Check infra: docker-compose.yml, Dockerfile, k8s/, terraform/
   - Check file patterns: *.tsx, *.py, *.go, *.rs, etc.
     ↓
2. Detect services
   - Match each directory against service profiles
   - Score each match (files found + deps matched + patterns matched)
   - Build composite detection result:
     {
       "detected_services": [
         { "name": "frontend", "path": "./frontend", "profile": "frontend-react", "score": 95 },
         { "name": "backend", "path": "./backend", "profile": "backend-node", "score": 90 },
         { "name": "database", "path": "./", "profile": "database-postgres", "score": 85 }
       ]
     }
     ↓
3. Match composition profile (or build composite)
   - Check compositions/ for matching combination
   - If no exact match, build composite from individual service profiles
     ↓
4. Present team recommendation to user
   - Show detected services
   - Show recommended team with always/recommended/optional tiers
   - User confirms or customizes
     ↓
5. Generate agents
   - Read base templates from devtools/agents/_bases/
   - Apply service profile customizations
   - Replace all {{PLACEHOLDERS}}
   - Write to project's .claude/agents/
     ↓
6. Write team-config.json
   - Detected services with paths and profiles
   - Generated agents list with timestamps
   - Phase configuration
   - Project state snapshot (dependency versions, file counts)
     ↓
7. Confirm to user
```

### Re-Run — Audit with Quality Scoring

```
/assemble-team (agents already exist)
     ↓
1. Scan current project state
     ↓
2. Score each existing agent (0-100%):

   Scoring criteria:
   ├── Framework match (25%)     — agent references correct framework/version?
   ├── File coverage (25%)       — agent knows about files it should work with?
   ├── Dependency awareness (20%) — agent knows current deps?
   ├── Convention alignment (15%) — agent follows project's actual patterns?
   └── Completeness (15%)        — agent has all required sections?
     ↓
3. Compare against team-config.json for drift:
   - New services added?
   - Dependencies updated?
   - New file patterns appeared?
   - Services removed?
     ↓
4. Report with scores:

   Agent Health Report:
   ✓ api-reviewer.md                — 95% (healthy)
   ⚠ react-frontend-developer.md  — 72% (BELOW 80% — REGENERATE)
      Missing: React 19 patterns, new component structure
   ✓ node-backend-developer.md     — 88% (healthy)

   New services detected:
   + Redis cache → needs cache-developer agent

   Actions:
   [A] Apply all (regenerate stale + add new)
   [S] Select which to apply
   [R] Full regenerate ALL from scratch
   [K] Keep current, no changes
     ↓
5. Apply:
   - Agents ≥ 80%: patch with minor updates
   - Agents < 80%: full regeneration from base template
   - New services: generate fresh agent
     ↓
6. Update team-config.json with new scores and state
```

---

## 8. `/orchestrate` Command — Phased Workflow

### Entry

```
/orchestrate <task description>
/orchestrate --phases 4,5,6 <task>     # Skip to specific phases
/orchestrate --resume                   # Resume interrupted run
```

### Workflow Pattern Selection

The orchestrator skill selects the pattern based on task complexity:

| Complexity | Detection | Pattern |
|-----------|-----------|---------|
| Simple | Single file change, bug fix, small tweak | Developer → Tester → Reviewer |
| Medium | Single service, moderate scope | Requirements → Developer + Tester (parallel) → Reviewer |
| Complex | Multi-service or large scope | Full 6-phase workflow |
| Plan exists | `docs/superpowers/plans/*` matches task | Skip to Phase 4 |
| Spec exists | `docs/superpowers/specs/*` matches task | Skip to Phase 3 |

### Phase Execution

```
Phase 1: Discovery (parallel)
├── Requirements Analyst → outputs: requirements.md
└── Researcher → outputs: research-report.md
    ↓
  GATE: Controller reviews outputs, confirms scope
    ↓
Phase 2: Design (parallel)
├── Architect → outputs: architecture.md
└── UX Designer → outputs: ux-spec.md (if UI work)
    ↓
  GATE: Cross-verification (see Section 8)
    ↓
Phase 3: Planning
└── Controller → invokes superpowers:writing-plans
    → outputs: implementation plan
    ↓
  GATE: User approves plan
    ↓
Phase 4: Implementation (file-scope isolated parallelism)
├── Developer A → writes code (scoped to ./frontend/ only)
├── Developer B → writes code (scoped to ./backend/ only)
│   NOTE: Multiple developers run in parallel ONLY if scoped to
│   different service directories. Same-scope devs run sequentially.
├── Tester → runs AFTER developers complete (sequential)
└── DevOps → infra changes (parallel, scoped to infra files only)
    ↓
  GATE: Cross-verification (see Section 8)
    ↓
Phase 5: Verification (parallel)
├── Reviewer → code review
├── Security Analyst → security scan
└── Cross-verification pairs
    ↓
  GATE: All verifiers pass (≥80% score)
    If fails → routes back to Phase 4 with feedback
    Max 3 retry loops, then surfaces to user
    ↓
Phase 6: Completion
├── Documentation Writer → generates/updates docs
└── Controller → final report + invokes superpowers:finishing-a-development-branch
```

### Existing Work Detection

Before starting, the manager scans for existing work:

```
Scan order:
1. Previous run? (.claude/orchestrator/runs/*<task>*)
   → Resume from last completed phase

2. Spec exists? (docs/superpowers/specs/*<task>*)
   → Skip Phase 1-2, start at Phase 3

3. Plan exists? (docs/superpowers/plans/*<task>*)
   → Skip Phase 1-3, start at Phase 4

4. Partial implementation? (git log, changed files matching task)
   → Resume Phase 4 from detected progress

5. Nothing found
   → Start from Phase 1
```

### Run Log

Each run produces `.claude/orchestrator/runs/<run-name>/run-log.md`:

```markdown
# Run: Add user authentication with OAuth2
**Started:** 2026-03-15 14:30
**Status:** In Progress — Phase 4

## Phase 1: Discovery ✓
- Requirements Analyst: DONE (requirements.md)
- Researcher: DONE (research-report.md)
- Gate: PASSED (manager approved scope)

## Phase 2: Design ✓
- Architect: DONE (architecture.md)
- UX Designer: SKIPPED (no UI work)
- Gate: PASSED (cross-verify score: 92%)

## Phase 3: Planning ✓
- Plan: docs/superpowers/plans/2026-03-15-auth-oauth2.md
- Gate: PASSED (user approved)

## Phase 4: Implementation ← CURRENT
- node-backend-developer: IN PROGRESS
- fullstack-tester: IN PROGRESS (parallel)
- devops: NOT STARTED
```

---

## 9. Cross-Verification Matrix

### Phase 2 → Phase 3 Gate (1 verifier dispatch)

A single **Architect-Reviewer** agent verifies design coherence:

| Check | What it verifies |
|-------|-----------------|
| Requirements coverage | All requirements have corresponding architecture components? |
| Technical feasibility | UX spec is feasible with chosen architecture? |
| Security posture | Architecture has auth/authz? No obvious security gaps? |

This is 1 subagent dispatch, not 4 separate ones — keeps context efficient.

### Phase 4 → Phase 5 Gate (1 verifier dispatch)

A single **Integration-Verifier** agent checks cross-service coherence:

| Check | What it verifies |
|-------|-----------------|
| API contracts | Frontend calls match backend endpoint signatures? |
| Test coverage | All code paths have tests? Tests pass? |
| Deployability | Env vars set? No hardcoded secrets? Config complete? |

### Phase 5 Verification (2 verifier dispatches — parallel, read-only)

Following the proven superpowers 2-reviewer pattern:

| Verifier | Checks |
|----------|--------|
| **Reviewer** (code quality) | Code quality, patterns, DRY, readability, implementation matches design |
| **Security Analyst** (security) | OWASP top 10, injection, auth bypass, data exposure, dependency vulnerabilities |

Both are read-only agents — safe to run in parallel.

### Developer Feedback Loop

If verification fails:
1. Controller invokes `superpowers:receiving-code-review` skill
2. Developer agent receives specific feedback
3. Developer fixes issues
4. Re-run verification (max 3 retry loops)
5. If still failing, surface to user

### Gate Scoring

```
Each verifier gives:
  PASS  = 100 points
  WARN  = 50 points  (proceed with logged warnings)
  FAIL  = 0 points   (blocks progress)

Gate score = average of all verifier scores

≥ 80%  → PROCEED to next phase
50-79% → PROCEED WITH WARNINGS (logged in run-log.md)
< 50%  → BLOCK — route back to previous phase with specific feedback
         Max 3 retry loops, then surface to user for decision
```

---

## 10. `/agent` Command — Standalone Usage

Use individual agents without full orchestration:

```
/agent <agent-name> <task>
/agent <agent1>,<agent2> <task>       # Multiple agents in parallel
/agent --list                          # Show available agents
```

### Behavior

1. **Read agent file** from `.claude/agents/<name>.md` (project-specific)
   - If not found and agents not yet generated: prompt "No project agents found. Run `/assemble-team` first to generate project-specific agents, or use `--generic` flag for a basic agent without project context."
   - With `--generic` flag: use base template but strip all `{{PLACEHOLDER}}` lines and replace with a generic instruction: "Analyze the current project context before proceeding."

2. **Single agent:** dispatch as subagent with the task

3. **Multiple agents:** dispatch all in parallel, collect results

4. **Output:** agent writes results to stdout (small tasks) or files (large outputs)

### Examples

```bash
# Single agent
/agent researcher "what auth libraries work best with Express.js?"
/agent security "scan the /api/auth endpoints for vulnerabilities"

# Multiple parallel agents
/agent developer,tester "add input validation to the signup form"

# List available agents
/agent --list
# Output:
#   Project agents (.claude/agents/):
#     react-frontend-developer  — React frontend specialist
#     node-backend-developer    — Node.js backend specialist
#     fullstack-tester          — Cross-service test writer
#
#   Base agents (available with --generic flag):
#     requirements, researcher, architect, developer, tester, ...
```

---

## 11. Superpowers Integration

The orchestrator integrates with existing superpowers skills at specific phase boundaries:

| Phase | Superpowers Skill | How |
|-------|-------------------|-----|
| Phase 2 → 3 | `superpowers:brainstorming` | If no spec exists, manager can invoke brainstorming before planning |
| Phase 3 | `superpowers:writing-plans` | Manager invokes to create implementation plan |
| Phase 4 | `superpowers:executing-plans` | Each developer follows the plan steps |
| Phase 4 | `superpowers:test-driven-development` | Tester follows TDD workflow |
| Phase 4 | `superpowers:subagent-driven-development` | Manager uses subagent pattern for parallel dev |
| Phase 5 | `superpowers:requesting-code-review` | Reviewer follows review checklist |
| Phase 5 | `superpowers:receiving-code-review` | Developer agents process review feedback in retry loop |
| Phase 5 | `superpowers:verification-before-completion` | All verifiers run verification checks |
| Phase 6 | `superpowers:finishing-a-development-branch` | Manager invokes to complete the work |

The manager agent decides which superpowers skills to invoke based on the task and context.

---

## 12. team-config.json Format

Written to `.claude/team-config.json` in the target project:

```json
{
  "project_name": "my-app",
  "generated_at": "2026-03-15T14:30:00Z",
  "last_audited": "2026-03-15T14:30:00Z",
  "detected_services": [
    {
      "name": "frontend",
      "path": "./frontend",
      "profile": "frontend-react",
      "score": 95,
      "dependencies": { "react": "^19.0.0", "vite": "^6.0.0" }
    },
    {
      "name": "backend",
      "path": "./backend",
      "profile": "backend-node",
      "score": 90,
      "dependencies": { "express": "^5.0.0", "prisma": "^6.0.0" }
    }
  ],
  "agents": [
    {
      "name": "react-frontend-developer",
      "file": ".claude/agents/react-frontend-developer.md",
      "base": "developer",
      "service": "frontend",
      "health_score": 92,
      "last_scored": "2026-03-15T14:30:00Z"
    }
  ],
  "phase_config": {
    "skip_ux": false,
    "skip_devops": false,
    "parallel_dev_test": true
  },
  "project_snapshot": {
    "file_count": 234,
    "dependency_hash": "a1b2c3d4",
    "last_commit": "abc1234"
  }
}
```

---

## 13. Communication Between Agents

### Large Outputs — Files

Each agent writes structured outputs to the run directory:

```
.claude/orchestrator/runs/<run-name>/
├── requirements.md          ← Requirements Analyst
├── research-report.md       ← Researcher
├── architecture.md          ← Architect
├── ux-spec.md              ← UX Designer
├── verification-report.md   ← Reviewer
├── security-report.md       ← Security Analyst
├── test-report.md          ← Tester
├── devops-report.md        ← DevOps
├── docs-output.md          ← Documentation Writer
└── run-log.md              ← Manager (phase tracking)
```

### Small Handoffs — Context Injection

The controller passes summaries between phases:

```
Phase 1 → Phase 2:
  Controller reads requirements.md + research-report.md
  Generates 3-5 sentence summary
  Injects into Architect + UX Designer prompts

Phase 2 → Phase 3:
  Controller reads architecture.md + cross-verify results
  Passes key decisions to planning phase

Phase 4 → Phase 5:
  Controller summarizes what was implemented + test results
  Injects into Reviewer + Security Analyst prompts
```

This keeps context windows efficient — full reports stay in files, only summaries travel between agents.

---

## 14. Audit Scoring Implementation

The audit is performed by the **controller** (top-level session) using the orchestrator skill, not a separate agent. The scoring process:

1. **Read the generated agent file** (e.g., `.claude/agents/react-frontend-developer.md`)
2. **Scan the project** for current state:
   - Read `package.json` / manifest for current dependency versions
   - Glob for file patterns the agent should know about
   - Read `CLAUDE.md` for current conventions
3. **Compare** each scoring dimension:
   - **Framework match (25%):** Does agent mention the correct framework version? (e.g., "React 19" vs actual "React 19.1")
   - **File coverage (25%):** Does agent reference directories/patterns that exist? Are there new directories it doesn't know about?
   - **Dependency awareness (20%):** Does agent mention current key dependencies?
   - **Convention alignment (15%):** Does agent's instructions match project's CLAUDE.md?
   - **Completeness (15%):** Does agent have all required sections (Task, Rules, Tech Stack, etc.)?
4. **Score** is calculated as a weighted percentage
5. **Result** written to `team-config.json` per agent

This is a heuristic check, not a test suite. The controller reads files and makes judgments — no specialized tooling needed.

---

## 15. Resume Mechanism

### How `/orchestrate --resume` works

1. Controller reads `.claude/orchestrator/runs/` to find the most recent run
2. Reads `run-log.md` from that run directory
3. Parses the phase completion markers (`✓` = done, `← CURRENT` = in progress)
4. Reconstructs state:
   - Completed phases → skip
   - Current phase → read the phase's output files for context
   - Generate summary of completed work from output files
5. Resume from the current phase with that summary as context

### If run-log is incomplete

- If `run-log.md` is missing or corrupted, fall back to scanning output files:
  - `requirements.md` exists → Phase 1 done
  - `architecture.md` exists → Phase 2 done
  - Plan file exists → Phase 3 done
  - Git log shows implementation commits → Phase 4 partially done
- Present findings to user: "Recovered state: Phases 1-3 complete, Phase 4 partial. Resume from Phase 4?"

---

## 16. Future Extensibility

| When | Action |
|------|--------|
| New framework support | Add service profile in `_profiles/services/` |
| New agent role needed | Add base template in `_bases/`, update roster |
| Custom workflow pattern | Add pattern to orchestrator skill |
| Team templates | Add composition profile in `_profiles/compositions/` |
| Per-project agent customization | Edit generated agents in `.claude/agents/` directly |
