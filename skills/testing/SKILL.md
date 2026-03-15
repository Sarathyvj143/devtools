---
name: testing
description: Use when writing or running tests — comprehensive testing workflow with positive/negative scenarios, coverage enforcement, and multi-service coordination
---

# Testing Skill

Comprehensive testing workflow that all tester agents follow. This skill ensures consistent, thorough testing across all services.

## Pre-Testing: Discovery Phase

Before writing ANY tests, always do this first:

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
- Test utilities/helpers already created
- Mocking patterns in use
- Coverage configuration if exists

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

## Test Writing Process

1. **Check existing tests** — don't duplicate, extend
2. **Follow existing patterns** — use same structure, utilities, mocking approach
3. **Write test plan first** — list all scenarios before writing code
4. **Write failing tests** — TDD red phase
5. **Group by feature** — not by type (all login tests together, not all unit tests together)
6. **Name descriptively** — `should return 401 when token is expired` not `test auth`

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
