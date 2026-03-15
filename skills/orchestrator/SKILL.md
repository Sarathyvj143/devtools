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

**Step 4a: Start services**
- Dispatch Dev Runner → starts all services in dependency order
- Dispatch Health Monitor → begins watching service health
- Dispatch Log Tracker → begins capturing logs
- Wait for Dev Runner to confirm all services healthy

**Step 4b: Implement**
- Dispatch developers in parallel ONLY if scoped to different service directories
- Same-scope developers run sequentially
- DevOps runs in parallel if scoped to infra-only files
- Each developer follows `superpowers:executing-plans`

**Step 4c: Test (per-service testers in parallel, then integration)**
- All tester agents invoke `devtools:testing` skill first
- Per-service testers run in parallel (scoped to their own service):
  - Frontend tester → frontend tests (components, UI, accessibility)
  - Backend tester → backend tests (API, middleware, business logic)
  - Database tester → database tests (queries, migrations, integrity)
  - Cloud tester → infrastructure tests (IaC validation, security compliance)
- AFTER per-service testers complete: Integration tester runs
  - Reads per-service test reports
  - Runs cross-service E2E and contract tests
  - Correlates failures with Log Tracker output
- Health Monitor reports any service degradation during tests
- All testers update project test scripts (package.json, etc.)

**Gate:** Dispatch Integration-Verifier agent to check:
- API contracts match between services
- All tests pass
- Health Monitor reports all services healthy
- Log Tracker reports no critical errors

### Phase 5: Verification (parallel — read-only agents)

**Step 5a: Production verification**
- Dispatch Prod Runner → builds and starts production mode
- Health Monitor → validates production health (5-minute sustained check)
- Log Tracker → captures production logs for errors

**Step 5b: Code verification (parallel — read-only)**
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
- Stop Dev Runner / Prod Runner services
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
