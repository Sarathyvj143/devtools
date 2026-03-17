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

### Step 6: Detect AI Tools and External Assistants

Before generating agents, detect which AI tools are available:

```bash
# Gemini CLI
GEMINI_AVAILABLE="false"
if which gemini > /dev/null 2>&1; then
  GEMINI_AVAILABLE="true"
  GEMINI_VERSION=$(gemini --version 2>/dev/null || echo "unknown")
  echo "Gemini CLI: available ($GEMINI_VERSION)"
fi

# Check for Gemini extensions
ls ~/.gemini/extensions/ 2>/dev/null

# Other AI tools
which cursor > /dev/null 2>&1 && echo "Cursor: available"
which copilot > /dev/null 2>&1 && echo "Copilot: available"

# Design MCP servers
grep -i "figma\|design\|image\|browser\|screenshot" .mcp.json ~/.claude/.mcp.json 2>/dev/null
```

**Store results in team-config.json:**
```json
{
  "ai_tools": {
    "gemini": { "available": true, "version": "2.0.0" },
    "cursor": { "available": false },
    "design_mcp": { "available": false }
  }
}
```

**When Gemini is available, bake it into ONLY frontend/UI agents:**
- **UX Designer** → add Gemini design planning (user flows, component structure, responsive layouts, accessibility review)
- **Frontend Developer** → add Gemini implementation planning (component tree from UX spec, responsive layout)

**NOT added to:** backend developer, database tester, devops, security, architect, etc.

**When Gemini is NOT available:**
- Frontend/UX agents work without it — all Gemini sections have fallback instructions
- No impact on functionality, just less AI assistance for UI work

**Frontend developer template selection:**
- If frontend service detected AND Gemini available → use `developer-frontend.md` (has Gemini integration)
- If frontend service detected AND no Gemini → use `developer-frontend.md` (Gemini sections have fallbacks)
- For non-frontend services → always use generic `developer.md` (no Gemini)

### Step 6b: Detect Actual Project Commands

Scan the project to discover the **real commands, ports, and paths** that will be baked into agents. This happens ONCE during assembly — generated agents get concrete commands, not detection logic.

#### 6a: For each service, discover:

**Package manager:**
```
Check service directory for lockfile:
  pnpm-lock.yaml → pnpm
  yarn.lock      → yarn
  bun.lockb      → bun
  package-lock.json → npm
  Pipfile → pipenv
  poetry.lock → poetry
  requirements.txt → pip
  go.mod → go
```

**Start command (dev mode):**
```
Node: Read package.json scripts → find "dev" or "start:dev" or "start"
      Example result: "pnpm run dev"
Python: Check for manage.py → "python manage.py runserver"
        Check deps for fastapi → "uvicorn app.main:app --reload --port 8000"
        Check deps for flask → "python -m flask run --port=8000 --reload"
Go: Find cmd/ directory → "go run ./cmd/server"
Docker: Read docker-compose.yml → "docker compose up <service>"
```

**Port:**
```
Read .env or .env.development for PORT=
Read docker-compose.yml for port mappings
Read package.json scripts for --port flag
Read framework defaults: React/Vite=5173, Next=3000, Express=3000, Flask=5000, FastAPI=8000
```

**Health endpoint:**
```
Grep codebase for: /health, /healthz, /api/health, health check routes
If found: use that exact path
If not found: use "/" (root)
```

**Test command:**
```
Node: Read package.json scripts.test → "pnpm run test"
Python: Check for pytest in deps → "python -m pytest tests/ -v"
Go: "go test ./... -v"
```

**Working directory:**
```
Absolute path to the service directory
Example: ./frontend, ./backend, ./services/auth
```

**Virtual environment (Python only):**
```
Check for: venv/, .venv/, env/, .env/
Activation: source <path>/bin/activate (Linux) or source <path>/Scripts/activate (Windows)
```

**Dependencies install command:**
```
Node: "<pkg-manager> install"
Python: "source venv/bin/activate && pip install -r requirements.txt"
Go: "go mod download"
```

#### 6b: Build startup order

From the detected services and their `depends_on` in profiles:
```
Example result:
  1. postgres (docker compose up -d postgres) — port 5432
  2. redis (docker compose up -d redis) — port 6379
  3. backend (cd backend && source venv/bin/activate && python -m flask run --port=8000) — port 8000
  4. frontend (cd frontend && pnpm run dev) — port 5173
```

#### 6c: Build health check commands per service

```
Example result:
  postgres: docker exec postgres pg_isready -U postgres
  redis: docker exec redis redis-cli ping
  backend: curl -s -o /dev/null -w "%{http_code}" http://localhost:8000/api/health
  frontend: curl -s -o /dev/null -w "%{http_code}" http://localhost:5173
```

### Step 7: Generate Agents with Concrete Commands

For each selected agent:
1. Read base template from `<plugin-path>/agents/_bases/<name>.md`
2. Read profile context for the agent's assigned service
3. Apply `agent_customizations` from the profile (name_prefix, extra_instructions)
4. Replace all `{{PLACEHOLDERS}}` with **actual values discovered in Step 6**:
   - `{{PROJECT_NAME}}` — from git remote URL or directory name
   - `{{TECH_STACK}}` — from profile `context` fields
   - `{{PROJECT_STRUCTURE}}` — run `ls` on relevant directories
   - `{{CONVENTIONS}}` — read CLAUDE.md if exists + profile conventions
   - `{{SERVICE_NAME}}` / `{{SERVICE_PATH}}` — from detection result (actual path)
   - `{{TEST_RUNNER}}` — actual test command discovered (e.g., "pnpm run test")
   - `{{TEST_COMMANDS}}` — concrete test run commands (e.g., "cd backend && python -m pytest tests/ -v")
   - `{{DATABASE_TYPE}}` — detected database type (e.g., "PostgreSQL", "MongoDB")
   - `{{STARTUP_COMMANDS}}` — concrete service startup commands for dev-runner
   - `{{PRODUCTION_COMMANDS}}` — concrete build/start commands for prod-runner
   - `{{SERVICES_UNDER_TEST}}` — list of all services for integration tester
   - `{{SERVICE_TEST_INSTRUCTIONS}}` — merged tester instructions from profiles
   - `{{AI_TOOLS}}` — detected AI tools and their availability (Gemini, Cursor, design MCP)
   - Cloud-specific: `{{AWS_SERVICES}}`, `{{CLOUD_PROVIDERS}}`, `{{INFRA_PATHS}}`, etc.
   - NOTE: Do NOT replace `{{OUTPUT_DIR}}` — testers resolve the run directory at runtime using `ls -td .claude/orchestrator/runs/*/`
5. Write to `.claude/agents/<generated-name>.md`

#### Special Handling: Dev Runner

The generated dev-runner agent gets **concrete commands**, not detection logic:

```markdown
## Startup Commands (detected by /assemble-team)

### Service: postgres
- Type: database
- Start: `docker compose up -d postgres`
- Health: `docker exec postgres pg_isready -U postgres`
- Port: 5432
- Order: 1 (start first)

### Service: backend (Flask)
- Type: backend
- Working dir: `./backend`
- Pre-start: `cd backend && source venv/bin/activate && pip install -r requirements.txt`
- Start: `python -m flask run --host=0.0.0.0 --port=8000 --reload`
- Health: `curl -s http://localhost:8000/api/health`
- Port: 8000
- Order: 2 (after postgres)
- Depends on: postgres

### Service: frontend (React/Vite)
- Type: frontend
- Working dir: `./frontend`
- Pre-start: `cd frontend && pnpm install`
- Start: `pnpm run dev`
- Health: `curl -s http://localhost:5173`
- Port: 5173
- Order: 3 (after backend)
- Depends on: backend
```

The agent just reads these and executes them in order — no guessing.

#### Special Handling: Prod Runner

Same approach — concrete production commands:

```markdown
## Production Commands (detected by /assemble-team)

### Service: backend (Flask)
- Working dir: `./backend`
- Build: `cd backend && source venv/bin/activate && pip install -r requirements.txt`
- Start (Linux): `gunicorn -w 4 -b 0.0.0.0:8000 app:app`
- Start (Windows): `waitress-serve --port=8000 app:app`
- Health: `curl -s http://localhost:8000/api/health`

### Service: frontend (React/Vite)
- Working dir: `./frontend`
- Build: `cd frontend && pnpm run build`
- Start: `npx serve -s dist -l 3000`
- Health: `curl -s http://localhost:3000`
```

#### Special Handling: Per-Service Testers

Each tester gets the actual test commands for their service:

```markdown
## Test Commands (detected by /assemble-team)

- Working dir: `./backend`
- Activate env: `source venv/bin/activate`
- Run all: `python -m pytest tests/ -v`
- Run with coverage: `python -m pytest tests/ -v --cov=src --cov-report=term`
- Run unit only: `python -m pytest tests/unit/ -v`
- Run integration: `python -m pytest tests/integration/ -v`
```

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

### Step 8: Write team-config.json

Write `.claude/team-config.json` with:
- `project_name` — from git or directory
- `generated_at` — current timestamp
- `detected_services` — list with paths, profiles, scores, dependencies, **discovered commands**
- `agents` — list with name, file path, base template, service assignment, health score
- `phase_config` — from composition profile
- `project_snapshot` — file count, dependency hash, last commit
- `commands` — all discovered commands per service (start, test, build, health check, port)
- `log_dir` — path pattern for service logs: `.claude/logs/<timestamp>/`

Example `commands` section in team-config.json:
```json
{
  "log_dir_pattern": ".claude/logs",
  "commands": {
    "postgres": {
      "start": "docker compose up -d postgres",
      "health": "docker exec postgres pg_isready -U postgres",
      "port": 5432,
      "order": 1
    },
    "backend": {
      "working_dir": "./backend",
      "pkg_manager": "pip",
      "venv_activate": "source venv/bin/activate",
      "install": "pip install -r requirements.txt",
      "start_dev": "python -m flask run --host=0.0.0.0 --port=8000 --reload",
      "start_prod_linux": "gunicorn -w 4 -b 0.0.0.0:8000 app:app",
      "start_prod_windows": "waitress-serve --port=8000 app:app",
      "test": "python -m pytest tests/ -v",
      "test_coverage": "python -m pytest tests/ -v --cov=src --cov-report=term",
      "health": "curl -s http://localhost:8000/api/health",
      "port": 8000,
      "order": 2,
      "depends_on": ["postgres"]
    },
    "frontend": {
      "working_dir": "./frontend",
      "pkg_manager": "pnpm",
      "install": "pnpm install",
      "start_dev": "pnpm run dev",
      "build": "pnpm run build",
      "start_prod": "npx serve -s dist -l 3000",
      "test": "pnpm run test",
      "test_coverage": "pnpm run test -- --coverage",
      "health": "curl -s http://localhost:5173",
      "port": 5173,
      "order": 3,
      "depends_on": ["backend"]
    }
  }
}
```

This `commands` block is the source of truth. Dev runner, prod runner, and testers all read from it.

### Step 9: Confirm

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
