# FET Critical Tests — Mock Execution Guide

**Tests:** FET-006, FET-007, FET-008  
**Method:** Mock MCP responses (deterministic, repeatable, no live Jira needed)

---

## Mock Implementation Strategy

Instead of hitting live Jira with timeouts/404s (risky, slow), we use **fixture data** and **conditional logic** in Fetcher.

### Option A: Fixture-Based (Simplest)

Create pre-saved raw JSON files that represent API responses:

```
test-harness/cat12-fetcher/fixtures/
├── FET-006-timeout-response.json
├── FET-007-404-response.json
└── FET-008-429-response.json
```

Modify `source-config.md` for test run to point to fixture directory.  
Fetcher reads fixtures instead of calling real MCP tool.

**Pros:** Simple, deterministic, fast  
**Cons:** Requires code path change to use fixtures

### Option B: MCP Mock Wrapper (Robust)

Create wrapper that intercepts MCP calls:

```
test-harness/cat12-fetcher/mcp-mock-wrapper.md
```

When Fetcher calls `mcp_atlassian-mcp_getJiraIssue`:
- Check if story key matches test pattern (e.g., `H20-TIMEOUT-TEST`, `H20-DELETED-404`)
- If match: return mock response (timeout delay, 404, 429)
- If no match: pass through to real MCP tool

**Pros:** No code changes, can mix real + mock calls  
**Cons:** More complex to implement

---

## Quick Start: Option A (Fixture-Based)

### Step 1: Create Test Fixtures

**File:** `test-harness/cat12-fetcher/fixtures/FET-006-timeout-response.json`

```json
{
  "error": "TIMEOUT",
  "message": "Request timeout after 60 seconds",
  "story_key": "H20-TIMEOUT-TEST"
}
```

**File:** `test-harness/cat12-fetcher/fixtures/FET-007-404-response.json`

```json
{
  "error": "NOT_FOUND",
  "message": "Issue does not exist",
  "story_key": "H20-DELETED-404",
  "valid_stories": ["H20-VALID-001", "H20-VALID-002"]
}
```

**File:** `test-harness/cat12-fetcher/fixtures/FET-008-429-response.json`

```json
{
  "error": "RATE_LIMITED",
  "message": "Too Many Requests",
  "retry_after_seconds": 60,
  "stories": ["H20-RATE-001", "H20-RATE-002", ..., "H20-RATE-010"],
  "rate_limit_hit_at_call": 101
}
```

### Step 2: Create Test Config

**File:** `test-harness/cat12-fetcher/source-config-mock.md`

```markdown
# Source Config (Mock Mode)

story_source: fixture
fixture_path: test-harness/cat12-fetcher/fixtures/
project_key: THE

Mock Mode: When source=fixture, Fetcher reads from fixture files instead of live Jira.
```

### Step 3: Run Test

```
1. Copy source-config-mock.md → {PROJECT_OUTPUT}/config/source-config.md
2. Load @fetcher agent
3. Provide story IDs matching fixture keys:
   - FET-006: "H20-TIMEOUT-TEST"
   - FET-007: "H20-VALID-001, H20-DELETED-404, H20-VALID-002"
   - FET-008: "H20-RATE-001, H20-RATE-002, ..., H20-RATE-010"
4. Fetcher reads fixture and simulates response
5. Observe behavior per test expectations
```

---

## Execution Plan

| Test | Mock Response | Expected Behavior | Verification |
|------|---------------|-------------------|--------------|
| **FET-006** | TIMEOUT after 60s | No file written, error shown, retry offered | No raw file exists; registry empty |
| **FET-007** | 404 at story #2 | Story #1 saved, batch halted, user prompted | Story #1 raw file exists; story #2 not saved |
| **FET-008** | 429 at call #101 | Pause 60s, resume, all 10 stories fetched | All 10 raw files exist; registry updated |

---

## Alternative: Live Jira Simulation

If fixture-based is too complex, alternative:

**Manual Test (Slow but Real):**
1. Create test story in Jira with unique key (e.g., H20-TIMEOUT-TEST)
2. Throttle network to Jira (Charles Proxy / browser DevTools)
3. Fetcher calls → network delays response 70+ seconds
4. Observe timeout behavior

**Drawback:** Requires network manipulation, slower, less repeatable

---

## Recommended: Proceed with Option A

Use fixtures for speed + determinism. When ready for integration testing, use live Jira.

Next steps:
1. Create fixture files (3 JSON files, ~100 lines total)
2. Update source-config to support "fixture" mode
3. Run FET-006, 007, 008 with mocks
4. Log results to adversarial-testing.md

Ready to implement?

