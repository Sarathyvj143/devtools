---
name: orchestrator
description: Use when running /orchestrate to manage phased multi-agent workflows with cross-verification gates
---

# Orchestrator

Manages phased multi-agent workflows. This skill runs at the controller level — it is NOT a dispatched subagent. It instructs the top-level session how to dispatch agents, manage phases, and handle cross-verification.

## Prerequisites
- Project agents must exist in `.claude/agents/` (run `/assemble-team` first)
- Team config must exist at `.claude/team-config.json`

## Workflow Pattern Selection

Assess task complexity and select pattern:

| Complexity | Detection | Pattern |
|-----------|-----------|---------|
| Simple | Single file change, bug fix, small tweak | Developer → Tester → Reviewer |
| Medium | Single service, moderate scope | Requirements → Developer + Tester → Reviewer |
| Complex | Multi-service or large scope | Full 6-phase workflow |
| Plan exists | `docs/superpowers/plans/*` matches task | Skip to Phase 4 |
| Spec exists | `docs/superpowers/specs/*` matches task | Skip to Phase 3 |

## Existing Work Detection

Before starting, scan for existing work:
1. Previous run? (`.claude/orchestrator/runs/*`) → Resume from last completed phase
2. Spec exists? (`docs/superpowers/specs/*`) → Skip Phase 1-2
3. Plan exists? (`docs/superpowers/plans/*`) → Skip Phase 1-3
4. Nothing found → Start from Phase 1

## Phase Execution

### Phase 1: Discovery (parallel — read-only agents)
Dispatch in parallel:
- Requirements Analyst → writes `requirements.md`
- Researcher → writes `research-report.md`

**Gate:** Read both outputs. Confirm scope is clear and complete. If questions remain, ask user.

### Phase 2: Design (parallel — read-only agents)
Dispatch in parallel:
- Architect → writes `architecture.md`
- UX Designer → writes `ux-spec.md` (skip if no UI work)

Pass Phase 1 summaries as context to both agents.

**Gate:** Dispatch Architect-Reviewer agent to verify:
- All requirements have architecture components
- UX spec is technically feasible
- Security posture is adequate

### Phase 3: Planning
Invoke `superpowers:writing-plans` to create implementation plan.

**Gate:** User approves the plan.

### Phase 4: Implementation (file-scope isolated parallelism)
Read team-config.json for service assignments.
- Dispatch developers in parallel ONLY if scoped to different service directories
- Same-scope developers run sequentially
- Tester runs AFTER developers complete
- DevOps runs in parallel if scoped to infra-only files

Each developer follows `superpowers:executing-plans`.

**Gate:** Dispatch Integration-Verifier agent to check:
- API contracts match between services
- All tests pass
- Deployment config is valid

### Phase 5: Verification (parallel — read-only agents)
Dispatch in parallel:
- Reviewer → writes `verification-report.md`
- Security Analyst → writes `security-report.md`
- Cost Optimizer → writes `cost-report.md` (if cloud detected)

Both follow the superpowers 2-reviewer pattern.

**Gate scoring:**
- PASS (100) / WARN (50) / FAIL (0) per verifier
- Average >= 80% → proceed
- 50-79% → proceed with warnings
- < 50% → route back to Phase 4 (max 3 retries)

If verification fails, invoke `superpowers:receiving-code-review` for developer to process feedback.

### Phase 6: Completion
- Dispatch Documentation Writer → updates docs
- Invoke `superpowers:finishing-a-development-branch`
- Write final run-log.md summary

## Run Log Management

Create `.claude/orchestrator/runs/<date>-<task-name>/run-log.md` at start.
Update after each phase completes with status markers:
- `✓` = completed
- `← CURRENT` = in progress
- Phase details: agent name, output file, gate result

## Resume Support

When `--resume` is used:
1. Find most recent run in `.claude/orchestrator/runs/`
2. Read `run-log.md` for phase completion markers
3. If run-log incomplete, scan for output files to determine state
4. Resume from last incomplete phase with summary context

## Communication Between Phases

- **Large outputs:** Agents write to files in run directory
- **Small handoffs:** Controller reads output files, generates 3-5 sentence summary, injects into next phase agents' prompts
- Full reports stay in files — only summaries travel between agents
