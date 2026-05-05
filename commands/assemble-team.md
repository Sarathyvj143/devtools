---
description: Detect project type, generate project-specific agents, or audit existing team
argument-hint: [--regenerate]
allowed-tools: [Read, Write, Edit, Glob, Grep, Bash]
---

# Assemble Team

$ARGUMENTS

## Instructions

Analyze the current project, detect services and technologies, and generate a team of project-specific agents.

### Platform Notes

- Bash examples are provided for Linux/macOS and Windows with Git Bash/WSL.
- For Windows PowerShell users without bash, use the PowerShell equivalents below.

#### PowerShell Equivalents (Step 1)

```powershell
# Get current plugin version (from devtools plugin install path)
$pluginPath = python -c "import json; d=json.load(open(r'$env:USERPROFILE\\.claude\\plugins\\installed_plugins.json')); print([v[0]['installPath'] for k,v in d.items() if 'devtools' in k][0])"
$pluginVersion = python -c "import json; print(json.load(open(r'$pluginPath\\.claude-plugin\\plugin.json'))['version'])"
$pluginCommit = (git -C $pluginPath rev-parse HEAD) 2>$null
$pluginDate = (git -C $pluginPath log -1 --format=%ci) 2>$null
"Plugin version: $pluginVersion"
"Plugin commit: $pluginCommit"
"Plugin updated: $pluginDate"

# Read team-config.json for when agents were generated
if (Test-Path .claude\\team-config.json) {
  $c = Get-Content .claude\\team-config.json | ConvertFrom-Json
  "Agents generated with: $($c.plugin_version) $($c.plugin_commit) $($c.generated_at)"
}
```

```powershell
# Detecting what changed in the plugin (PowerShell)
$oldCommit = "<from team-config.json plugin_commit>"
$newCommit = (git -C $pluginPath rev-parse HEAD)
$changedFiles = (git -C $pluginPath diff --name-only $oldCommit $newCommit) 2>$null
$changedBases = $changedFiles | Where-Object { $_ -like "agents/_bases/*" }
$changedProfiles = $changedFiles | Where-Object { $_ -like "agents/_profiles/*" }
$changedSkills = $changedFiles | Where-Object { $_ -like "skills/*" }
$changedCommands = $changedFiles | Where-Object { $_ -like "commands/*" }
```

### Step 1: Check for Existing Team and Plugin Version

Check if `.claude/agents/` and `.claude/team-config.json` exist.

#### 1a: Check Plugin Version

```bash
# Get current plugin version (from devtools plugin install path)
PLUGIN_PATH=$(python -c "
import json
d=json.load(open('$HOME/.claude/plugins/installed_plugins.json'))
for k,v in d.items():
  if 'devtools' in k:
    print(v[0]['installPath'])
" 2>/dev/null)

PLUGIN_VERSION=$(python -c "
import json
print(json.load(open('$PLUGIN_PATH/.claude-plugin/plugin.json'))['version'])
" 2>/dev/null)

# Get plugin's last update timestamp (git commit date)
PLUGIN_COMMIT=$(cd "$PLUGIN_PATH" && git rev-parse HEAD 2>/dev/null)
PLUGIN_DATE=$(cd "$PLUGIN_PATH" && git log -1 --format=%ci 2>/dev/null)

echo "Plugin version: $PLUGIN_VERSION"
echo "Plugin commit: $PLUGIN_COMMIT"
echo "Plugin updated: $PLUGIN_DATE"
```

#### 1b: Compare with Generated Agents

```bash
# Read team-config.json for when agents were generated
if [ -f .claude/team-config.json ]; then
  GENERATED_WITH=$(python -c "
import json
c=json.load(open('.claude/team-config.json'))
print(c.get('plugin_version','unknown'))
print(c.get('plugin_commit','unknown'))
print(c.get('generated_at','unknown'))
" 2>/dev/null)
  echo "Agents generated with: $GENERATED_WITH"
fi
```

#### 1c: Determine Action

| Situation | Action |
|-----------|--------|
| No agents exist | First run -> go to Step 2 |
| Plugin commit matches team-config -> project unchanged | "Team is up to date" -> audit only for project drift |
| Plugin commit CHANGED since generation | "Plugin updated! Base templates may have improved." -> recommend regenerate |
| Project changed (new deps, new files) | Audit and patch/regenerate as needed |
| `--regenerate` flag | Skip audit, regenerate all |
| `--update` flag | Regenerate only from new plugin templates, keep project context |

**If plugin updated since last generation:**
```
DevTools plugin updated since team was generated!
  Generated with: v1.0.0 (commit abc1234, 2026-03-15)
  Current plugin: v1.1.0 (commit def5678, 2026-03-20)

  Changes in plugin:
  - Updated tester templates with MCP integration
  - Added Gemini support for frontend developer
  - Fixed dev-runner log file handling

  Recommend: Regenerate agents to get latest improvements.
  [U] Update agents (regenerate from new templates + keep project context)
  [A] Audit only (check project drift, don't update templates)
  [R] Full regenerate from scratch
  [K] Keep current, no changes
```

**If agents exist, plugin unchanged** (re-run / audit mode):
1. Read `.claude/team-config.json` for previous state
2. Scan current project state (manifests, file patterns, dependencies)
3. Score each existing agent (0-100%) against current project:
   - Framework match (25%) -- correct framework/version referenced?
   - File coverage (25%) -- knows about current file patterns?
   - Dependency awareness (20%) -- references current dependencies?
   - Convention alignment (15%) -- matches project's CLAUDE.md?
   - Completeness (15%) -- has all required sections?
4. Detect new services not in previous config
5. Present health report:
   - Agents >= 80%: healthy (minor patch)
   - Agents < 80%: STALE -- will be fully regenerated
   - New services: new agents needed
6. Present actions: [A]pply all / [S]elect / [R]egenerate all / [K]eep
7. Apply selected changes and update team-config.json

If `$ARGUMENTS` contains `--regenerate`, skip audit and regenerate all from scratch.
If `$ARGUMENTS` contains `--update`, run the Update Flow below.

**If no agents exist** (first run):
Continue to Step 2.

### Step 1d: Update Flow (`--update`)

When plugin templates have changed, this is how agents get updated while preserving project context:

#### What gets preserved (from team-config.json):
- Detected services, paths, and ports
- Package managers and start/test/build commands
- Health check endpoints
- Dependency order
- AI tools availability
- All values from Step 6 (command detection)

#### What gets regenerated (from new plugin templates):
- Agent instructions, process, rules, checklists
- MCP integration sections
- Test categories and testing depth
- Log handling and shell session constraints
- Gemini/AI tool usage patterns

#### The actual update process:

```
For each generated agent in .claude/agents/:

  1. Read the generated agent file
  2. Extract project-specific sections (everything between ## markers that contains
     real project values -- tech stack, service path, startup commands, etc.)
  3. Read the NEW base template from plugin (e.g., agents/_bases/tester-backend.md)
  4. Check: has the base template actually changed?
     - Compare plugin's base template git hash vs what's recorded in team-config.json
     - If unchanged: skip this agent (no update needed)
     - If changed: continue to regenerate
  5. Regenerate: apply new base template + inject preserved project context
  6. Write updated agent to .claude/agents/<name>.md
  7. Log: "Updated <name>.md -- new template applied, project context preserved"
```

#### Detecting what changed in the plugin:

```bash
PLUGIN_PATH="<from team-config.json plugin_path>"
OLD_COMMIT="<from team-config.json plugin_commit>"
NEW_COMMIT=$(cd "$PLUGIN_PATH" && git rev-parse HEAD)

# List changed files between old and new plugin versions
cd "$PLUGIN_PATH"
CHANGED_FILES=$(git diff --name-only "$OLD_COMMIT" "$NEW_COMMIT" 2>/dev/null)

# Categorize changes
CHANGED_BASES=$(echo "$CHANGED_FILES" | grep "agents/_bases/" || true)
CHANGED_PROFILES=$(echo "$CHANGED_FILES" | grep "agents/_profiles/" || true)
CHANGED_SKILLS=$(echo "$CHANGED_FILES" | grep "skills/" || true)
CHANGED_COMMANDS=$(echo "$CHANGED_FILES" | grep "commands/" || true)
```

Show user what changed:
```
Plugin updated: v1.0.0 -> v1.1.0

Changed base templates:
  * agents/_bases/tester-backend.md (enhanced MCP integration)
  * agents/_bases/tester-frontend.md (added spec-driven testing)
  * agents/_bases/developer-frontend.md (added Gemini support)
  - agents/_bases/developer.md (unchanged)
  - agents/_bases/devops.md (unchanged)

Changed profiles:
  * agents/_profiles/services/backend-node.json (added runner commands)

Changed skills:
  * skills/testing/SKILL.md (added coverage thresholds)
  * skills/ai-design-assist/SKILL.md (new skill)

Your project agents affected:
  * node-backend-tester.md -> will be regenerated (base template changed)
  * react-frontend-tester.md -> will be regenerated (base template changed)
  * react-frontend-developer.md -> will be regenerated (base template changed + new Gemini support)
  - node-backend-developer.md -> no update needed (base template unchanged)
  - postgres-db-tester.md -> no update needed
  - integration-tester.md -> no update needed

Proceed? [Y]es / [N]o / [S]elect which to update
```

#### Handling user-modified agents:

Before overwriting any agent, check if the user manually edited it:

```bash
# Compare generated agent's header comment with what's on disk
# Each generated agent has a comment: # Generated by devtools v1.0.0 at 2026-03-15
# If the file was modified after generation (git shows changes), warn:

for agent in .claude/agents/*.md; do
  agent_name=$(basename "$agent")
  generated_at=$(python -c "
import json
agents = json.load(open('.claude/team-config.json')).get('agents',[])
for a in agents:
  if a['file'].endswith('$agent_name'):
    print(a.get('generated_at',''))
" 2>/dev/null)

  # Check if file was modified after generation
  file_modified=$(stat -c %Y "$agent" 2>/dev/null || stat -f %m "$agent" 2>/dev/null)
  # Compare timestamps...

  if [ file_was_manually_edited ]; then
    echo "WARNING: $agent_name was manually edited after generation"
    echo "  [O]verwrite with new template (lose manual edits)"
    echo "  [M]erge (keep manual edits, add new template sections)"
    echo "  [S]kip (keep current version)"
  fi
done
```

#### After update, write new team-config.json:
- Update `plugin_version` to current
- Update `plugin_commit` to current
- Update `generated_at` for each regenerated agent
- Keep all project context (services, commands, ports) unchanged

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
1. Check each profile's `detect.files` -- do these files exist?
2. Check `detect.dependencies` -- present in manifest?
3. Check `detect.patterns` -- do matching files exist?
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
  frontend (./frontend) -- React
  backend (./backend) -- Node.js/Express
  database -- PostgreSQL
  cloud -- AWS (CDK)

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
- **UX Designer** -> add Gemini design planning (user flows, component structure, responsive layouts, accessibility review)
- **Frontend Developer** -> add Gemini implementation planning (component tree from UX spec, responsive layout)

**NOT added to:** backend developer, database tester, devops, security, architect, etc.

**When Gemini is NOT available:**
- Frontend/UX agents work without it -- all Gemini sections have fallback instructions
- No impact on functionality, just less AI assistance for UI work

**Frontend developer template selection:**
- If frontend service detected AND Gemini available -> use `developer-frontend.md` (has Gemini integration)
- If frontend service detected AND no Gemini -> use `developer-frontend.md` (Gemini sections have fallbacks)
- For non-frontend services -> always use generic `developer.md` (no Gemini)

### Step 6b: Detect Actual Project Commands

Scan the project to discover the **real commands, ports, and paths** that will be baked into agents. This happens ONCE during assembly -- generated agents get concrete commands, not detection logic.

#### 6a: For each service, discover:

**Package manager:**
```
Check service directory for lockfile:
  pnpm-lock.yaml -> pnpm
  yarn.lock      -> yarn
  bun.lockb      -> bun
  package-lock.json -> npm
  Pipfile -> pipenv
  poetry.lock -> poetry
  requirements.txt -> pip
  go.mod -> go
```

**Start command (dev mode):**
```
Node: Read package.json scripts -> find "dev" or "start:dev" or "start"
      Example result: "pnpm run dev"
Python: Check for manage.py -> "python manage.py runserver"
        Check deps for fastapi -> "uvicorn app.main:app --reload --port 8000"
        Check deps for flask -> "python -m flask run --port=8000 --reload"
Go: Find cmd/ directory -> "go run ./cmd/server"
Docker: Read docker-compose.yml -> "docker compose up <service>"
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
Node: Read package.json scripts.test -> "pnpm run test"
Python: Check for pytest in deps -> "python -m pytest tests/ -v"
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
  1. postgres (docker compose up -d postgres) -- port 5432
  2. redis (docker compose up -d redis) -- port 6379
  3. backend (cd backend && source venv/bin/activate && python -m flask run --port=8000) -- port 8000
  4. frontend (cd frontend && pnpm run dev) -- port 5173
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

#### Platform note (v2b)

For Gemini CLI installs, read base templates from `<plugin-path>/agents/_bases-gemini/<name>.md` instead. Those files are generated from `agents/_bases/` by `scripts/build-gemini-agents.py` — same content, but with frontmatter shaped for Gemini's subagent loader (`allowed-tools` renamed to `tools`, tool names mapped via `skills/using-devtools/references/gemini-tools.md`). On Claude Code and Codex, continue using `agents/_bases/`. If the user passes `--platform=gemini` (or running on Gemini host), select `_bases-gemini/`. Auto-detect by checking `$env:GEMINI_CLI` or the absence of `~/.claude/`.

Note: a full Gemini `/assemble-team` flow also needs a Gemini-side equivalent of `.claude/team-config.json` (e.g. `.gemini/team-config.json`) and a way to run the generated subagents. Both are open follow-ups beyond v2b's foundation work.

#### Standard generation

For each selected agent:
1. Read base template from `<plugin-path>/agents/_bases/<name>.md`
2. Read profile context for the agent's assigned service
3. Apply `agent_customizations` from the profile (name_prefix, extra_instructions)
4. Replace all `{{PLACEHOLDERS}}` with **actual values discovered in Step 6**:
   - `{{PROJECT_NAME}}` -- from git remote URL or directory name
   - `{{TECH_STACK}}` -- from profile `context` fields
   - `{{PROJECT_STRUCTURE}}` -- run `ls` on relevant directories
   - `{{CONVENTIONS}}` -- read CLAUDE.md if exists + profile conventions
   - `{{SERVICE_NAME}}` / `{{SERVICE_PATH}}` -- from detection result (actual path)
   - `{{TEST_RUNNER}}` -- actual test command discovered (e.g., "pnpm run test")
   - `{{TEST_COMMANDS}}` -- concrete test run commands (e.g., "cd backend && python -m pytest tests/ -v")
   - `{{DATABASE_TYPE}}` -- detected database type (e.g., "PostgreSQL", "MongoDB")
   - `{{STARTUP_COMMANDS}}` -- concrete service startup commands for dev-runner
   - `{{PRODUCTION_COMMANDS}}` -- concrete build/start commands for prod-runner
   - `{{SERVICES_UNDER_TEST}}` -- list of all services for integration tester
   - `{{SERVICE_TEST_INSTRUCTIONS}}` -- merged tester instructions from profiles
   - `{{AI_TOOLS}}` -- detected AI tools and their availability (Gemini, Cursor, design MCP)
   - Cloud-specific: `{{AWS_SERVICES}}`, `{{CLOUD_PROVIDERS}}`, `{{INFRA_PATHS}}`, etc.
   - NOTE: Do NOT replace `{{OUTPUT_DIR}}` -- testers resolve the run directory at runtime using `ls -td .claude/orchestrator/runs/*/`
5. **Add generation header** to the top of every generated agent (before the frontmatter):
   ```markdown
   <!-- Generated by devtools v1.1.0 (commit abc1234) at 2026-03-17T14:30:00Z -->
   <!-- Base template: developer.md | Profile: backend-node | Service: backend -->
   <!-- DO NOT manually edit above the --- frontmatter line. Run /assemble-team --update to refresh. -->
   ```
6. Write to `.claude/agents/<generated-name>.md`

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

The agent just reads these and executes them in order -- no guessing.

#### Special Handling: Prod Runner

Same approach -- concrete production commands:

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
- If Playwright/Puppeteer MCP found -> add to frontend tester's available tools
- If Database MCP found -> add to database tester's available tools
- If API Client MCP found -> add to integration tester's available tools
- Add discovered MCP tools to each tester's prompt as `{{MCP_TOOLS}}`

**Test script updates:**
Each tester agent is responsible for updating the project's test scripts (package.json, pyproject.toml, Makefile) to include commands for their test category.

### Step 8: Write team-config.json

Write `.claude/team-config.json` with:
- `project_name` -- from git or directory
- `generated_at` -- current timestamp
- **`plugin_version`** -- devtools plugin version (e.g., "1.0.0")
- **`plugin_commit`** -- git commit hash of the plugin when agents were generated
- **`plugin_path`** -- path to devtools plugin installation
- `detected_services` -- list with paths, profiles, scores, dependencies, **discovered commands**
- `agents` -- list with name, file path, base template, service assignment, health score
- `phase_config` -- from composition profile
- `project_snapshot` -- file count, dependency hash, last commit
- `ai_tools` -- detected AI tools (Gemini, etc.) and availability
- `commands` -- all discovered commands per service (start, test, build, health check, port)
- `log_dir` -- path pattern for service logs: `.claude/logs/<timestamp>/`

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

Example `agents` section in team-config.json (tracks per-agent generation state):
```json
{
  "agents": [
    {
      "name": "react-frontend-developer",
      "file": ".claude/agents/react-frontend-developer.md",
      "base_template": "developer-frontend.md",
      "profile": "frontend-react",
      "service": "frontend",
      "base_template_hash": "a1b2c3d4",
      "generated_at": "2026-03-17T14:30:00Z",
      "health_score": 95,
      "manually_edited": false
    },
    {
      "name": "node-backend-tester",
      "file": ".claude/agents/node-backend-tester.md",
      "base_template": "tester-backend.md",
      "profile": "backend-node",
      "service": "backend",
      "base_template_hash": "e5f6g7h8",
      "generated_at": "2026-03-17T14:30:00Z",
      "health_score": 92,
      "manually_edited": false
    }
  ]
}
```

The `base_template_hash` field enables smart updates:
- During `--update`, compute current hash of each base template in the plugin
- Compare against stored hash in team-config.json
- If hash changed -> this agent needs regeneration
- If hash unchanged -> skip (no update needed)

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
