# FET Critical Tests — Fetcher API Resilience
**Tests:** FET-006, FET-007, FET-008  
**Date Created:** 2026-06-22  
**Priority:** 🔴 CRITICAL (identified in coverage audit)

---

## FET-006: Jira API Timeout

**Scenario:** Jira API takes >60 seconds to respond (network latency, server overload)

**Setup:**
- Project: TEST-HARNESS-EPICS
- Story to fetch: H20-199 (real Jira)
- Precondition: Network delay or Jira rate limit triggers 60+ second wait

**Steps:**
1. Load `@fetcher` agent
2. Provide story IDs: `H20-199`
3. Agent makes Jira API call via MCP tool
4. Simulate: API does not respond for 70 seconds
5. Observe orchestrator behavior

**Expected Behavior:**
```
- After 60 seconds: timeout triggered
- Message shown: "Jira API timeout after 60s for story H20-199"
- User offered: "Retry? (yes / no)"
- On yes: Retry up to 3 times (with exponential backoff)
- On no: Halt batch, lock released, clear error message shown
- Registry NOT updated (no partial state)
```

**Actual Result:** ⏳ PENDING EXECUTION

---

## FET-007: Partial Batch Failure

**Scenario:** Batch of 3 stories; story #2 returns 404 (deleted), others are valid

**Setup:**
- Project: TEST-HARNESS-EPICS
- Stories: H20-199 (exists), H20-DELETE-ME (deleted), H20-201 (exists)
- Precondition: H20-DELETE-ME removed from Jira after fetch was scheduled

**Steps:**
1. Load `@fetcher` agent
2. Provide story IDs: `H20-199, H20-DELETE-ME, H20-201`
3. Agent fetches H20-199 ✓
4. Agent tries H20-DELETE-ME → 404 Not Found
5. Observe behavior

**Expected Behavior:**
```
Story #1 (H20-199):
  - Fetched successfully
  - Raw file saved: stories/raw/H20-199.raw.json ✓
  - Registry updated: status="fetched" ✓

Story #2 (H20-DELETE-ME):
  - API returns 404: "Not Found"
  - **HALT ENTIRE BATCH** (do not fetch story #3)
  - Error shown: "Story H20-DELETE-ME not found (404). Batch processing halted."
  - User offered: "Skip H20-DELETE-ME and continue with remaining? (yes / no)"
  
  On yes:
  - Remove H20-DELETE-ME from batch
  - Resume with H20-201
  - Story #3 fetched
  
  On no:
  - Halt completely
  - No further stories fetched
  - Lock released

Story #3 (H20-201):
  - Only fetched if user selected "yes" at halt
  - If fetched: should succeed
```

**Actual Result:** ⏳ PENDING EXECUTION

---

## FET-008: Rate Limiting Recovery

**Scenario:** Jira rate limit = 100 calls/min; batch needs 150 calls

**Setup:**
- Project: TEST-HARNESS-EPICS
- Stories: 10 stories, each requiring ~15 API calls
- Precondition: Jira API configured with strict rate limit (100/min)

**Steps:**
1. Load `@fetcher` agent
2. Provide 10 story IDs
3. Agent makes API calls sequentially
4. At call #100: Jira responds with 429 (Too Many Requests)
5. Response includes `Retry-After: 60` header
6. Observe recovery behavior

**Expected Behavior:**
```
Calls 1-100:
  - All succeed
  - Stories 1-6 fetched completely (~90 calls)
  - Story #7 partially fetched (~10 calls)

Call #101 (attempting story #7):
  - Jira responds: 429 Too Many Requests
  - Header: Retry-After: 60
  - Agent detects: Rate limit hit
  - Message shown: "Rate limited by Jira API. Resuming after 60 second cooldown..."
  - Pipeline pauses for 60 seconds

After cooldown (61 seconds):
  - Agent resumes from where stopped
  - Continues fetching story #7 remaining calls
  - Fetches stories #8, #9, #10
  - All 10 stories eventually succeed
  - No data loss
  - No partial files left behind
  - Registry updated: all 10 stories status="fetched"

User notification:
  - At halt: "Rate limited. Pausing..."
  - At resume: "Rate limit cooldown complete. Resuming fetch..."
```

**Actual Result:** ⏳ PENDING EXECUTION

---

## How to Simulate

### FET-006 (Timeout)
- Use network throttling (browser DevTools / Charles Proxy)
- Set latency to 70,000ms (70 seconds) for Jira API endpoint
- Or: Mock MCP tool to delay response by 70s

### FET-007 (404 Not Found)
- Create fixture story ID that doesn't exist in Jira
- Or: Mock MCP getJiraIssue to return 404 for specific key

### FET-008 (Rate Limit 429)
- Mock MCP tool to:
  - Count calls
  - At call #100: return 429 with Retry-After header
  - After 60s: accept further calls

---

## Recording Results

Results logged to: `docs/adversarial-testing.md` → Category 12 (Fetcher)

Format:
```
| FET-006 | Jira timeout after 60s | [Description] | [Actual Result] | [Hardening] |
| FET-007 | Partial batch failure | [Description] | [Actual Result] | [Hardening] |
| FET-008 | Rate limiting 429 | [Description] | [Actual Result] | [Hardening] |
```

