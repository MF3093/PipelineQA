# Category 17 — Integration & Version Consistency (INT)

**Tests:** INT-001, INT-002, INT-003  
**Priority:** 🟡 HIGH (cross-sprint dependencies, version tracking)  
**Date:** 2026-06-22

---

## INT-001: Context Version Consistency

**Scenario:** Context is approved at version 1. Strategy is generated under context v1 and approved. Context is then updated to version 2. TC generation should warn that strategy is stale.

**Setup:**
1. Approve context: context_version = 1, context_approved_at = 2026-06-20 10:00:00
2. Approve strategy under this context: strategy_current_version = 1, strategy_approved_at = 2026-06-20 11:00:00
3. Update context: context_version = 2, context_approved_at = 2026-06-22 09:00:00
4. Start TC generation (which reads strategy state)

**Expected Behavior:**
```
Orchestrator Step 5 (CONTEXT/STRATEGY VERSION CHECK):
- Detects: strategy_approved_at < context_approved_at
- Alert shown: "Warning: the approved strategy (v1) was generated under context v1.
              Context has since been updated to v2.
              Strategy may not reflect current tech stack or client priorities.
              Recommend re-running Story Prioritizer before TC generation. Continue anyway? (yes / no)"
- On 'no': STOP. Release lock. Do not proceed to TC generation.
- On 'yes': warn but continue.
```

**Actual Result:** TBD

---

## INT-002: Story Dependency Chain Verification

**Scenario:** Story A has dependency on Story B. Story B is not yet parsed. TC Generator attempts to generate TCs for Story A.

**Setup:**
1. Create fetched-stories entries: Story-A (status=parsed, dependencies=[Story-B]), Story-B (status=fetched, not parsed yet)
2. Run TC Generator on Story-A
3. Check if dependency is validated before TC generation

**Expected Behavior:**
```
TC Generator Step 2 (Prerequisites):
- Reads Story-A.parsed.json
- Checks dependencies field: ["Story-B"]
- Looks up Story-B in fetched-stories.json: status = "fetched"
- Alert shown: "Story-A depends on Story-B, but Story-B has not been parsed yet.
               Parse Story-B first, then re-run TC generation for Story-A.
               Or mark dependency as 'unblocked' if not applicable to this batch."
- Halts TC generation until dependency is resolved.
```

**Actual Result:** TBD

---

## INT-003: Registry State Across Multiple Runs

**Scenario:** Run 1 fetches and parses stories. Run 2 (later in day) starts fresh and reads same project. Previously fetched stories should not be re-fetched.

**Setup:**
1. Run 1: Fetch stories A, B, C → all fetched_at = 2026-06-22 09:00:00
2. Close run. Release lock.
3. Run 2: Start new batch on same day
4. Check if fetched stories are in registry and marked as 'parsed' (not 're-fetched')

**Expected Behavior:**
```
Orchestrator Run 2:
- Reads fetched-stories.json
- Sees: A, B, C already have status='parsed'
- Does NOT re-fetch these stories (they're already in registry with parsed status)
- If user requests "Fetch only" for A, B, C: agent halts with
  "Stories A, B, C are already registered with status 'parsed'.
   Re-fetch will overwrite approved parsed output.
   Continue? (yes / no)"
- On 'no': stop, do not re-fetch.
- On 'yes': proceed with re-fetch (requires explicit approval).
```

**Actual Result:** TBD

---

## Execution Plan

| Test | Setup | Actual Result | Pass Criteria |
|------|-------|---|---|
| **INT-001** | Context v1 → v2, strategy remains v1, start TC gen | Warning shown if strategy approved before context update | Version mismatch detected, warning shown |
| **INT-002** | Story-A depends on Story-B (not yet parsed) | Dependency check halts TC gen, user alerted | Dependencies validated before TC generation |
| **INT-003** | Run 1 fetches A,B,C; Run 2 sees same stories already parsed | Orchestrator does not re-fetch without approval | Registry state preserved across runs |

---

## Recording Results

Log to: `docs/adversarial-testing.md` → Category 17 (Integration & Version Consistency)
