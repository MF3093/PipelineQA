# Category 11 — Data Integrity Tests (DIN)

**Tests:** DIN-002, DIN-003, DIN-004  
**Priority:** 🟡 HIGH (orphaned files, schema mismatch can corrupt pipeline state)  
**Date:** 2026-06-22

---

## DIN-002: Raw File Exists But Registry Empty

**Scenario:** File exists on disk but no entry in `fetched-stories.json` registry

**Setup:**
- Project: TEST-HARNESS-EPICS
- Precondition: Manually create `stories/raw/TEST-DIN-002.raw.json` (orphaned file)
- Registry: `fetched-stories.json` has NO entry for TEST-DIN-002

**Steps:**
1. Create raw file: `test-harness/project-output-epics/stories/raw/TEST-DIN-002.raw.json`
2. Add valid JSON content (minimal story data)
3. Verify: `fetched-stories.json` does NOT contain TEST-DIN-002
4. Run Orchestrator startup (reconciliation check)

**Expected Behavior:**
```
Startup reconciliation:
- Scans all raw files
- Finds TEST-DIN-002.raw.json
- Checks registry: NOT FOUND
- Alert shown: "Orphaned file detected: stories/raw/TEST-DIN-002.raw.json"
- Offers: "Add to registry as fetched? (yes / no / ignore)"

On yes:
- Add entry to fetched-stories.json
- status: "fetched"
- Parser can then process it

On no:
- Leave file orphaned
- Flag file in tracking/assumptions.md
- Continue run

On ignore:
- Skip this file
- Continue run
```

**Actual Result:** ⏳ PENDING EXECUTION

---

## DIN-003: TC CSV Exists But Not Registered

**Scenario:** TC file exists but no entry in `pipeline-state.json` registry for that story

**Setup:**
- Project: TEST-HARNESS-EPICS
- Story: TEST-DIN-003 (parsed, prioritized, TCs generated)
- Precondition: TC file exists, but registry `tc_approvals` doesn't contain TEST-DIN-003

**Steps:**
1. Verify: `test-cases/TEST-DIN-003-test-cases.csv` exists with TCs
2. Verify: `pipeline-state.json → tc_approvals` does NOT have entry for TEST-DIN-003
3. Run Orchestrator (any phase)

**Expected Behavior:**
```
Startup/phase transition:
- Scans registry for story state
- Checks for orphaned TC files (exist but not registered)
- Alert shown: "Orphaned TC file detected: TEST-DIN-003-test-cases.csv"
  Details: "Story status is 'parsed' but 150 TCs exist in test-cases/ folder"
- Options:
  1. "Register TCs as approved? (yes / no)"
  2. "Delete orphaned file? (yes / no)"
  3. "Ignore and continue?"

On register:
- Add entry to tc_approvals
- Set status: "tc_generated"
- TCs counted and logged

On delete:
- Remove TC file
- Registry unchanged
- Prevents duplicate generation

On ignore:
- Leave orphaned
- Flag in tracking/assumptions.md
- Risk: TCs might be re-generated, creating duplicates
```

**Actual Result:** ⏳ PENDING EXECUTION

---

## DIN-004: Cross-Agent Schema Mismatch

**Scenario:** Parser output doesn't match expected ParsedStory schema; downstream agent fails to read

**Setup:**
- Project: TEST-HARNESS-EPICS
- Story: TEST-DIN-004 (manually created parsed file with bad schema)
- Precondition: Corrupt parsed file missing required field

**Steps:**
1. Create `stories/parsed/TEST-DIN-004.parsed.json` with incomplete schema:
   - Missing: `acs[]` (required array)
   - Missing: `extraction_quality` (required field)
   - Has: `story_key`, `description`, other fields
2. Add entry to `fetched-stories.json`: status="parsed"
3. Try to run Story Analyzer on TEST-DIN-004

**Expected Behavior:**
```
Story Analyzer attempts to read parsed file:
- Reads TEST-DIN-004.parsed.json
- Validates against ParsedStory schema
- Detection: Missing required fields (acs, extraction_quality)
- Error shown: "Invalid ParsedStory schema for TEST-DIN-004
  Missing required fields: [acs, extraction_quality]
  File may be corrupted. Remediation options:"
  1. "Re-run Parser on TEST-DIN-004 (fix by re-parsing)"
  2. "Skip this story (continue with others)"
  3. "Halt and investigate manually"

On re-run:
- Story Analyzer halts
- User instructed to run Parser for TEST-DIN-004
- Parser regenerates correct schema

On skip:
- Story skipped
- Logged as DATA_ERROR in tracking/assumptions.md
- Pipeline continues with other stories

On halt:
- Pipeline pauses
- User must manually investigate/fix
```

**Actual Result:** ⏳ PENDING EXECUTION

---

## Execution Plan

| Test | Setup | Actual Result | Pass Criteria |
|------|-------|---------------|---------------|
| **DIN-002** | Create orphaned raw file | Alert shown, recovery offered | File detected, user given options |
| **DIN-003** | Create orphaned TC file | Alert shown, recovery offered | File detected, clean resolution |
| **DIN-004** | Create invalid parsed file | Schema validation fails, error shown | Mismatch detected, halt/skip/rerun |

---

## Recording Results

Log to: `docs/adversarial-testing.md` → Category 11 (Data Integrity)

Format:
```
| DIN-002 | Raw file orphaned | [Expected] | [Actual] | [Hardening] |
| DIN-003 | TC file orphaned | [Expected] | [Actual] | [Hardening] |
| DIN-004 | Schema mismatch | [Expected] | [Actual] | [Hardening] |
```

