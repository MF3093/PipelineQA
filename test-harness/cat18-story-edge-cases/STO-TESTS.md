# Category 18 — Story Content Edge Cases (STO)

**Tests:** STO-001, STO-002, STO-003, STO-004  
**Priority:** 🟡 HIGH (Parser robustness, TC Generator handling of edge cases)  
**Date:** 2026-06-22

---

## STO-001: Story with No Acceptance Criteria

**Scenario:** Raw Jira story has zero acceptance criteria. Parser processes it.

**Setup:**
1. Raw story: {"key": "STO-001", "summary": "Add search feature", "description": "Allow users to search", "acceptanceCriteria": []}
2. Parser processes story
3. Check parsed output and Story Analyzer behavior

**Expected Behavior:**
```
Parser Step 4 (Extraction):
- Extracts: key, summary, description
- ACs array: empty []
- Flags: ["NO_ACS"] or raises question in assumptions.md
- Parsed file written with empty acs array
- extraction_quality.sections.acs = "not_found" or "empty"

Story Analyzer Step 1b (Schema Validation):
- Validates: acs field exists (✓) but is empty array
- Question logged: "No acceptance criteria found for STO-001. 
                   Cannot generate test cases without defined expected behaviors.
                   Suggest: (a) Return to story author for AC definition,
                            (b) Skip story and mark as blocked"
```

**Actual Result:** TBD

---

## STO-002: Story with Circular Dependencies

**Scenario:** Story A depends on Story B, which depends on Story A (circular).

**Setup:**
1. Story-A: dependencies = ["Story-B"]
2. Story-B: dependencies = ["Story-A"]
3. Orchestrator attempts to order stories for TC generation

**Expected Behavior:**
```
Orchestrator Step 12 (Dependency Resolution or TC Generator prereq):
- Detects circular dependency: A → B → A
- Alert shown: "Circular dependency detected: Story-A ↔ Story-B.
               Cannot resolve execution order. 
               Mark one dependency as 'optional' or manually resolve."
- Offers: (a) Mark A→B as optional, (b) Mark B→A as optional, (c) Skip both
- Does NOT proceed with TC generation until resolved
```

**Actual Result:** TBD

---

## STO-003: Story with > 50 ACs

**Scenario:** Story has unusually high number of acceptance criteria (complexity edge case).

**Setup:**
1. Story with 75 acceptance criteria
2. Parser extracts all 75
3. Story Analyzer reviews; TC Generator creates test cases

**Expected Behavior:**
```
Parser:
- Extracts all 75 ACs successfully
- No truncation or loss of data

Story Analyzer:
- Question logged: "Story STO-003 has 75 acceptance criteria.
                   This is unusually high complexity. 
                   Recommend breaking into smaller stories or validating scope.
                   Risk: test case explosion may exceed practical limits."
- Continues analysis

TC Generator:
- Generates TCs for all 75 ACs (no artificial limit)
- May warn: "Generated [N] test cases from 75 ACs. Consider splitting story."
- All TCs written to CSV (no truncation)
```

**Actual Result:** TBD

---

## STO-004: Story with Special Characters & Unicode

**Scenario:** AC text contains special characters, Unicode emoji, HTML entities, regex-like syntax.

**Setup:**
1. AC text: "Validate email regex: ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ and show ✅/❌"
2. Another AC with HTML: "Display error: <div class='error'>Invalid input!</div>"
3. Parser processes story

**Expected Behavior:**
```
Parser Step 3 (Injection Scan):
- Scans AC text for injection patterns (SYSTEM:, <prompt>, [INST])
- HTML tags detected: <div class='error'> — flagged but not treated as injection
- Regex and emoji: legitimate content, not flagged

Parser Step 4 (Extraction):
- Extracts AC text as-is (literal, not interpreted)
- Preserves regex, emoji, HTML literally in parsed JSON

Story Analyzer:
- Reads parsed AC
- No interpretation of regex or HTML
- Question may note: "AC contains regex and HTML — ensure test framework handles escape sequences"

TC Generator:
- Writes AC text to CSV (escaped/quoted properly for CSV format)
- CSV output: "AC-1","Validate email regex: ^[a-zA-Z0-9._%+-]+@...","✅/❌" (properly quoted)
- No data loss, no execution of HTML or regex
```

**Actual Result:** TBD

---

## Execution Plan

| Test | Setup | Actual Result | Pass Criteria |
|------|-------|---|---|
| **STO-001** | Story with zero ACs | Parser extracts empty array; Analyzer flags question or blocker | Empty ACs handled gracefully, question logged |
| **STO-002** | Circular dependencies A↔B | Dependency cycle detected before TC gen; user alerted | Circular dependencies detected, pipeline halts |
| **STO-003** | Story with 75 ACs | All 75 parsed and TCs generated (no truncation) | High complexity handled, no data loss |
| **STO-004** | AC with regex, emoji, HTML | Special chars preserved in parsed JSON and CSV (properly escaped) | Special characters and Unicode handled safely |

---

## Recording Results

Log to: `docs/adversarial-testing.md` → Category 18 (Story Content Edge Cases)
