---
name: testing
description: Use when writing or running tests — comprehensive testing workflow with positive/negative scenarios, coverage enforcement, and multi-service coordination
---

# Testing Skill

Comprehensive testing workflow that all tester agents follow. This skill ensures consistent, thorough testing across all services.

## Pre-Testing: Discovery Phase

Before writing ANY tests, always do this first:

### 0. Understand What Was Implemented
**This is the most critical step.** Before you can test, you must know what was built.

Read these sources in order:
1. **Developer output** — `{{OUTPUT_DIR}}/developer-output.md` (what was implemented, which files changed)
2. **Git diff** — run `git diff HEAD~N` to see actual code changes
3. **Implementation plan** — check `docs/superpowers/plans/` for the task's plan
4. **Architecture doc** — check `{{OUTPUT_DIR}}/architecture.md` for design decisions
5. **New/modified files** — run `git diff --name-only HEAD~N` to get the file list

From these, build a **test map:**
```
Feature: User Registration
  Files changed:
    frontend/src/pages/Register.tsx        (new)
    frontend/src/components/RegisterForm.tsx (new)
    backend/src/routes/auth.ts             (modified — added POST /register)
    backend/src/services/user.service.ts   (new)
    backend/src/validators/auth.validator.ts (new)
    database/migrations/001_create_users.sql (new)

  API endpoints:
    POST /api/auth/register — { email, password, name } → { user, token }

  DB changes:
    users table — id, email, password_hash, name, created_at
```

This test map drives ALL test planning. Without it, you're guessing.

### 1. Scan Existing Tests
```
Find existing test files:
- **/*.test.{ts,tsx,js,jsx}
- **/*.spec.{ts,tsx,js,jsx}
- **/test_*.py, **/*_test.py
- **/*_test.go
- **/tests/**, **/__tests__/**
```

Understand:
- Test framework being used (jest, vitest, pytest, go test, etc.)
- Test patterns and conventions (file naming, describe/it structure, fixtures)
- Test utilities/helpers already created (test factories, custom matchers, fixtures)
- Mocking patterns in use (MSW, nock, unittest.mock, etc.)
- Coverage configuration if exists
- Test directory structure (co-located vs separate `tests/` dir)

### 2. Scan Test Scripts
Read project test configuration:
- `package.json` → scripts.test, scripts.test:unit, scripts.test:e2e, scripts.test:coverage
- `pyproject.toml` → [tool.pytest], [tool.coverage]
- `Makefile` → test targets
- `docker-compose.test.yml` → test infrastructure
- `.github/workflows/` → CI test commands
- `jest.config.*`, `vitest.config.*`, `pytest.ini`, `setup.cfg`

### 3. Check MCP Servers
If MCP servers are available, check for testing tools:
- Browser automation (Playwright, Puppeteer MCP)
- API testing (Postman, Insomnia MCP)
- Database testing (direct DB query MCP)
- Monitoring/observability (Datadog, Grafana MCP)

Use MCP tools when available instead of building from scratch.

### 4. Set Up Test Environment

Before running tests, ensure the test environment is ready:

**Test Database:**
- Check if a test database config exists (`.env.test`, `DATABASE_URL_TEST`)
- If not, create one: `<db_name>_test`
- Run migrations on the test database
- Each test should use transaction rollback for isolation (preferred) or truncate tables

**Test Data Seeding:**
- Check for existing seed scripts (`db:seed:test`, `fixtures/`, `factories/`)
- If none exist, create test data factories:
  - For Node: use `@faker-js/faker` or test factories
  - For Python: use `factory_boy` or `model_bakery`
  - For Go: use test helper functions
- Seed data should be minimal — only what's needed for the test

**Test Environment Variables:**
- Check for `.env.test` or test-specific config
- Ensure test env vars point to test database, not dev
- Ensure API URLs point to test server, not production

**Docker Test Infrastructure (if applicable):**
- Check for `docker-compose.test.yml`
- Start test infrastructure: `docker-compose -f docker-compose.test.yml up -d`

## Test Planning

For each feature/change, plan tests across ALL these categories:

### Positive Tests (Happy Path)
- Valid inputs produce expected outputs
- Correct status codes for valid requests
- Data is persisted correctly
- UI renders correctly with valid data
- Workflows complete successfully end-to-end

### Negative Tests (Unhappy Path)
- Invalid inputs are rejected with proper errors
- Missing required fields return 400/422
- Unauthorized access returns 401/403
- Non-existent resources return 404
- Malformed requests are handled gracefully
- Rate limiting works correctly
- Concurrent access is handled (race conditions)

### Boundary Tests
- Empty strings, null, undefined
- Maximum length inputs
- Minimum/maximum numeric values
- Special characters (unicode, emoji, SQL injection strings)
- Very large payloads
- Zero, negative numbers where positive expected

### Error Recovery Tests
- Network failures mid-operation
- Database connection drops
- Timeout handling
- Partial failures in multi-step operations
- Retry logic verification

### Security Tests
- SQL/NoSQL injection attempts
- XSS payload handling
- CSRF token validation
- Auth token expiration handling
- Privilege escalation attempts
- Sensitive data not leaked in errors

### Performance Tests (when applicable)
- Response time under normal load
- Response time with concurrent requests
- Memory usage under load
- Database query performance (N+1 detection)
- Bundle size (frontend)

### Accessibility Tests (frontend)
- Keyboard navigation
- Screen reader compatibility (ARIA labels)
- Color contrast
- Focus management

## Test Execution Order

Write and run tests in this order — faster tests first, slower tests last:

```
1. Unit tests          (fastest — no external deps)
   ├── Business logic
   ├── Utility functions
   ├── Validators
   └── Pure component rendering
          ↓
2. Integration tests   (moderate — mocked external deps)
   ├── API endpoint tests (with mocked DB)
   ├── Component tests (with mocked API)
   └── Middleware tests
          ↓
3. Database tests      (moderate — needs test DB)
   ├── Migration tests
   ├── Query tests
   └── Constraint tests
          ↓
4. Contract tests      (needs both services)
   ├── API request/response shape matching
   └── Shared type consistency
          ↓
5. E2E tests           (slowest — needs all services running)
   ├── Full user workflows
   ├── Cross-service data flow
   └── Auth flow end-to-end
          ↓
6. Security tests      (separate pass)
   ├── Injection attempts
   ├── Auth bypass
   └── Data leakage
          ↓
7. Performance tests   (separate pass, optional)
   ├── Response time benchmarks
   ├── N+1 query detection
   └── Load handling
          ↓
8. Accessibility tests (frontend, separate pass)
   ├── Keyboard navigation
   ├── Screen reader
   └── Color contrast
```

Stop at first failure within each tier. Fix before moving to next tier.

## Test Writing Process

1. **Read developer output** — know exactly what was implemented and what files changed
2. **Check existing tests** — don't duplicate, extend
3. **Follow existing patterns** — use same structure, utilities, mocking approach
4. **Write test plan first** — list ALL scenarios (positive + negative + boundary) before writing code
5. **Write failing tests** — TDD red phase
6. **Group by feature** — not by type (all registration tests together, not all unit tests together)
7. **Name descriptively** — `should return 401 when token is expired` not `test auth`
8. **Run after each test file** — don't write all tests then run; write a file, run, verify, next file

## Cross-Tester Coordination

When multiple tester agents run in parallel on the same project:

### Shared Knowledge
- **Frontend tester needs to know:** API endpoints and response shapes (read backend's API docs or developer output)
- **Backend tester needs to know:** What the frontend sends (read frontend developer's output)
- **Database tester needs to know:** What queries the backend runs (read backend code)
- **Integration tester needs to know:** All per-service test results (read their reports)

### Test File Ownership (prevents conflicts)
Each tester owns specific test directories:
- Frontend tester → `frontend/src/__tests__/` or `frontend/tests/`
- Backend tester → `backend/tests/` or `backend/src/__tests__/`
- Database tester → `tests/db/` or `backend/tests/db/`
- Integration tester → `tests/integration/` or `tests/e2e/`
- Cloud tester → `infra/tests/` or `tests/infra/`

**Never write tests outside your owned directory.**

### Test Script Updates (prevents package.json conflicts)
Only ONE tester should update each service's package.json:
- Frontend tester updates `frontend/package.json`
- Backend tester updates `backend/package.json`
- If monorepo with single package.json: integration tester does the final merge of all test scripts
- Each tester writes their scripts to `{{OUTPUT_DIR}}/<tester>-scripts.json`
- Integration tester reads all and merges into the project's config

## Test Script Management

After writing tests, update project test scripts:

### Node/React (package.json)
Ensure these scripts exist:
```json
{
  "scripts": {
    "test": "vitest run",
    "test:watch": "vitest",
    "test:coverage": "vitest run --coverage",
    "test:unit": "vitest run --dir src/tests/unit",
    "test:integration": "vitest run --dir src/tests/integration",
    "test:e2e": "playwright test"
  }
}
```

### Python (pyproject.toml)
Ensure test config exists:
```toml
[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = "-v --cov=src --cov-report=html --cov-report=term"

[tool.coverage.run]
source = ["src"]
omit = ["tests/*"]

[tool.coverage.report]
fail_under = 80
```

### Go
Ensure Makefile targets:
```makefile
test:
	go test ./... -v -race
test-coverage:
	go test ./... -coverprofile=coverage.out
	go tool cover -html=coverage.out
```

## Coverage Enforcement

| Metric | Minimum Threshold |
|--------|------------------|
| Line coverage | 80% |
| Branch coverage | 70% |
| Function coverage | 85% |
| Critical paths | 100% (auth, payment, data mutation) |

After running tests, check coverage:
1. Run coverage command
2. Parse coverage report
3. If below threshold, identify uncovered lines
4. Write tests for uncovered critical paths first
5. Report coverage per service (for multi-service projects)

## Feature-Based Test Coordination

When testing a feature that spans multiple services (e.g., "user login"):

```
Feature: User Login
├── Frontend Tests (react-tester)
│   ├── Login form renders correctly
│   ├── Form validation (empty email, invalid format)
│   ├── Submit sends correct API request
│   ├── Success → redirect to dashboard
│   ├── Failure → show error message
│   └── Loading state during API call
├── Backend Tests (node-tester)
│   ├── POST /api/auth/login with valid credentials → 200 + JWT
│   ├── POST /api/auth/login with wrong password → 401
│   ├── POST /api/auth/login with nonexistent user → 401
│   ├── Rate limiting after 5 failed attempts → 429
│   └── JWT token contains correct claims
├── Database Tests (db-tester)
│   ├── User record is fetched correctly
│   ├── Failed login attempts are tracked
│   ├── Session is created on successful login
│   └── Index on email column is used (query plan)
└── Integration Tests (fullstack-tester)
    ├── End-to-end login flow (frontend → backend → DB → response)
    ├── Login with expired session → redirect to login page
    └── Concurrent login from multiple devices
```

The orchestrator coordinates: each service tester runs their part, then the fullstack integration tester validates the cross-service flow.

## MCP Integration

When MCP servers are available, use them:

| MCP Server | Use For |
|-----------|---------|
| Playwright/Puppeteer | E2E browser testing, visual regression |
| Database MCP | Direct DB assertions, test data seeding |
| API Client MCP | API contract testing, request/response validation |
| Monitoring MCP | Performance metrics during test execution |

Check `.mcp.json` and `~/.claude/.mcp.json` for available servers.

## Output Format

Write results to: `{{OUTPUT_DIR}}/test-report.md`

```markdown
# Test Report — [Feature Name]

## Summary
- Total: 45 tests
- Passed: 42
- Failed: 2
- Skipped: 1
- Coverage: 87% (lines), 78% (branches)

## Tests by Category
### Positive (20 tests) — 20 passed
### Negative (15 tests) — 13 passed, 2 failed
### Boundary (5 tests) — 5 passed
### Security (3 tests) — 3 passed
### Performance (2 tests) — 1 passed, 1 skipped

## Failed Tests
### test_login_rate_limiting
- Expected: 429 after 5 attempts
- Actual: 200 (rate limiter not triggered)
- Root cause: Rate limiter middleware not applied to /api/auth/login
- Log correlation: No errors in backend logs

## Coverage Report
| Service | Lines | Branches | Functions |
|---------|-------|----------|-----------|
| frontend | 89% | 76% | 91% |
| backend | 85% | 72% | 88% |
| Overall | 87% | 74% | 89% |

## Test Scripts Updated
- Added `test:e2e` to package.json
- Updated coverage threshold in vitest.config.ts

## Gaps
- Performance tests skipped (load testing tool not available)
```
