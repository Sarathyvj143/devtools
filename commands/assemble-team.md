---
description: Detect project type, generate project-specific agents, or audit existing team
argument-hint: [--regenerate]
allowed-tools: [Read, Write, Edit, Glob, Grep, Bash]
---

# Assemble Team

$ARGUMENTS

## Instructions

Analyze the current project, detect services and technologies, and generate a team of project-specific agents.

### Step 1: Check for Existing Team

Check if `.claude/agents/` and `.claude/team-config.json` exist.

**If agents exist** (re-run / audit mode):
1. Read `.claude/team-config.json` for previous state
2. Scan current project state (manifests, file patterns, dependencies)
3. Score each existing agent (0-100%) against current project:
   - Framework match (25%) — correct framework/version referenced?
   - File coverage (25%) — knows about current file patterns?
   - Dependency awareness (20%) — references current dependencies?
   - Convention alignment (15%) — matches project's CLAUDE.md?
   - Completeness (15%) — has all required sections?
4. Detect new services not in previous config
5. Present health report:
   - Agents >= 80%: healthy (minor patch)
   - Agents < 80%: STALE — will be fully regenerated
   - New services: new agents needed
6. Present actions: [A]pply all / [S]elect / [R]egenerate all / [K]eep
7. Apply selected changes and update team-config.json

If `$ARGUMENTS` contains `--regenerate`, skip audit and regenerate all.

**If no agents exist** (first run):
Continue to Step 2.

### Step 2: Scan Project Structure

Walk project root + subdirectories (max 3 levels deep):
- Check manifest files: `package.json`, `requirements.txt`, `go.mod`, `Cargo.toml`, `pyproject.toml`, `pom.xml`
- Check infrastructure: `docker-compose.yml`, `Dockerfile`, `k8s/`, `terraform/`, `*.tf`
- Check cloud: `cdk.json`, `serverless.yml`, `firebase.json`, `.azure/`, `app.yaml`
- Check file patterns: `*.tsx`, `*.py`, `*.go`, `*.rs`, `*.vue`, etc.

### Step 3: Match Service Profiles

Find the devtools plugin install path from `~/.claude/plugins/installed_plugins.json`.
Read all profiles from `<plugin-path>/agents/_profiles/services/*.json`.

For each directory in the project:
1. Check each profile's `detect.files` — do these files exist?
2. Check `detect.dependencies` — present in manifest?
3. Check `detect.patterns` — do matching files exist?
4. For cloud profiles, also check `detect.terraform_provider` in `*.tf` files
5. Score each match (files + deps + patterns found)
6. Keep matches with score > 0

Build composite detection result.

### Step 4: Select Composition

Check `<plugin-path>/agents/_profiles/compositions/*.json` for a matching combination.
If no exact match, use `general.json` as fallback.

### Step 5: Present Team Recommendation

Show user:
```
Detected services:
  frontend (./frontend) — React
  backend (./backend) — Node.js/Express
  database — PostgreSQL
  cloud — AWS (CDK)

Recommended team (N agents):
  react-frontend-developer
  node-backend-developer
  fullstack-tester
  reviewer
  devops
  cost-optimizer

Optional (add any?):
  architect
  ux-designer
  security-analyst
  docs-writer
  researcher
  requirements-analyst

Proceed? (y/customize)
```

Wait for user confirmation.

### Step 6: Generate Agents

For each selected agent:
1. Read base template from `<plugin-path>/agents/_bases/<name>.md`
2. Read profile context for the agent's assigned service
3. Apply `agent_customizations` from the profile (name_prefix, extra_instructions)
4. Replace all `{{PLACEHOLDERS}}`:
   - `{{PROJECT_NAME}}` — from git remote URL or directory name
   - `{{TECH_STACK}}` — from profile `context` fields
   - `{{PROJECT_STRUCTURE}}` — run `ls` on relevant directories
   - `{{CONVENTIONS}}` — read CLAUDE.md if exists + profile conventions
   - `{{SERVICE_NAME}}` / `{{SERVICE_PATH}}` — from detection result
   - `{{TEST_RUNNER}}` — from profile context
   - `{{OUTPUT_DIR}}` — `.claude/orchestrator/runs/<current-run>/`
   - Cloud-specific: `{{AWS_SERVICES}}`, `{{CLOUD_PROVIDERS}}`, `{{INFRA_PATHS}}`, etc.
5. Write to `.claude/agents/<generated-name>.md`

#### Special Handling: Tester Agents (Per-Service)

Instead of one generic tester, generate **multiple specialized tester agents** based on detected services:

**For each detected service, generate a specialized tester:**

| Service Type | Base Template | Generated Agent Name |
|-------------|---------------|---------------------|
| Frontend (React/Vue/Next) | `tester-frontend.md` | `react-frontend-tester.md` |
| Backend (Node/Python/Go) | `tester-backend.md` | `node-backend-tester.md` |
| Database (Postgres/Mongo) | `tester-database.md` | `postgres-db-tester.md` |
| Cloud (AWS/GCP/Azure) | `tester-cloud.md` | `aws-cloud-tester.md` |

**Always generate an integration tester for multi-service projects:**
- Base template: `tester-integration.md`
- Generated name: `integration-tester.md`
- Gets `{{SERVICES_UNDER_TEST}}` with ALL services listed
- Coordinates results from all per-service testers

**For single-service projects:**
- Generate one per-service tester (e.g., `react-frontend-tester.md`)
- Use the generic `tester.md` base template as fallback if no specialized template matches

**All tester agents must invoke the `devtools:testing` skill** before writing tests.

**Tester generation process:**
1. For each detected service, pick the matching tester base template
2. Apply profile's `agent_customizations.tester` (name_prefix, extra_instructions)
3. Replace placeholders with service-specific values
4. Replace `{{SERVICE_TEST_INSTRUCTIONS}}` with profile's `tester.extra_instructions`
5. Write to `.claude/agents/<name>-tester.md`
6. If multi-service: also generate `integration-tester.md` with all services listed

**MCP integration:**
- Check `.mcp.json` and `~/.claude/.mcp.json` for testing MCP servers
- If Playwright/Puppeteer MCP found → add to frontend tester's available tools
- If Database MCP found → add to database tester's available tools
- If API Client MCP found → add to integration tester's available tools
- Add discovered MCP tools to each tester's prompt as `{{MCP_TOOLS}}`

**Test script updates:**
Each tester agent is responsible for updating the project's test scripts (package.json, pyproject.toml, Makefile) to include commands for their test category.

### Step 7: Write team-config.json

Write `.claude/team-config.json` with:
- `project_name` — from git or directory
- `generated_at` — current timestamp
- `detected_services` — list with paths, profiles, scores, dependencies
- `agents` — list with name, file path, base template, service assignment, health score
- `phase_config` — from composition profile
- `project_snapshot` — file count, dependency hash, last commit

### Step 8: Confirm

Tell user:
```
Team assembled! N agents generated in .claude/agents/
Run /orchestrate <task> to start the team on a task.
Run /agent <name> <task> to use an agent standalone.
```

### Cloud Agent Rules

- Single cloud detected: enhance DevOps agent with cloud context (no separate cloud agent)
- Multiple clouds detected: generate separate cloud agents + Cost Optimizer
- Terraform/Pulumi detected: always generate infra agent
- Cost Optimizer is always generated when any cloud is detected
