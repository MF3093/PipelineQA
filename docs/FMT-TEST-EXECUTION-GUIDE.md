# Parser Format Coverage Tests — Execution Guide

**Test Series:** FMT-001 to FMT-011  
**Agent Under Test:** @parser  
**Test Project:** TEST-HARNESS-EPICS  
**Fixtures Location:** `test-harness/parser-formats/`  
**Project Output Location:** `test-harness/project-output-epics/`

---

## Setup Status ✓

✅ All 10 fixture files copied to `test-harness/project-output-epics/stories/raw/`  
✅ Registry entries added to `fetched-stories.json` with status="fetched"  
✅ Test environment ready

---

## Execution Order

Run tests in this sequence to ensure proper results documentation:

1. **FMT-001** — Plain text section headers
2. **FMT-002** — Bold `**Section**` headers  
3. **FMT-003** — Markdown `## Section` with subsections
4. **FMT-004** — Informal/no structure (single paragraph)
5. **FMT-006** — User story triple only (no ACs)
6. **FMT-007** — ACs as "As a / I want / so that" lines
7. **FMT-008** — Gherkin/BDD `Given/When/Then` format
8. **FMT-009** — Numbered list without AC header label
9. **FMT-010** — ADF JSON description (Jira Cloud)
10. **FMT-011** — HTML description (Azure DevOps/older Jira)

*Note: FMT-005 (mixed batch) will be run after FMT-001 and FMT-004 are documented*

---

## Test Execution Steps

### For Each Test (FMT-00N):

```
1. Open GitHub Copilot Chat
2. Load @parser
3. Say: "Parse story TEST-FMT-00N"
4. Wait for parser to complete
5. Verify output by checking:
   - {PROJECT_OUTPUT}/stories/parsed/TEST-FMT-00N.parsed.json exists
   - Content matches expectations below
6. Record results in Execution Results section (see below)
```

---

## Expected Behaviors Per Test

### FMT-001: Plain Text Section Headers

**Description Format:**
```
As a dispatcher I want to search and filter the aircraft list...

Acceptance Criteria:
1. The search field accepts text input...
2. The filter panel allows filtering...
3. When no results match the filter criteria...
4. Clearing all filters restores the full list...

Out of Scope:
Bulk operations on filtered results are not in scope...

Specified Behavior:
The search operates on the aircraft registration number...

Dependencies:
Requires the aircraft reference catalog...
```

**Expected Parsed Output:**
- ✅ `acs[]` = 4 entries (one per numbered item)
- ✅ `sections[]` contains "Specified Behavior" and "Dependencies"
- ✅ `out_of_scope[]` = ["Bulk operations on filtered results are not in scope"]
- ✅ `dependencies[]` = ["TEST-FMT-002"] (extracted from Dependencies section)
- ✅ No ad-hoc fields like `specified_behavior` or `dependencies_text`
- ✅ `needs_review` = false
- ✅ All ACs marked `testable: true`

**CRITICAL CHECK:**
```json
{
  "key": "TEST-FMT-001",
  "title": "Aircraft list — search and filter",
  "description": "As a dispatcher I want to search and filter...",
  "acs": [
    { "id": 1, "text": "The search field accepts text input...", "testable": true, "conditional": false },
    { "id": 2, "text": "The filter panel allows filtering...", "testable": true, "conditional": false },
    { "id": 3, "text": "When no results match the filter criteria...", "testable": true, "conditional": false },
    { "id": 4, "text": "Clearing all filters restores the full aircraft list.", "testable": true, "conditional": false }
  ],
  "sections": [
    { "header": "Specified Behavior", "content": "The search operates on the aircraft registration number..." },
    { "header": "Dependencies", "content": "Requires the aircraft reference catalog..." }
  ],
  "out_of_scope": ["Bulk operations on filtered results are not in scope for this story."],
  "dependencies": ["TEST-FMT-002"],
  "needs_review": false,
  "flags": []
}
```

---

### FMT-002: Bold `**Section**` Headers

**Description Format:**
```
As a dispatcher I want to view the full registration details...

**Acceptance Criteria**
1. The detail view displays the aircraft registration number...
2. The operational status badge reflects the current status...
3. If the aircraft has an active maintenance record...
4. If no active maintenance record exists...

**Out of Scope**
Editing aircraft registration data is not in scope...

**Specified Behavior**
The operational status values are: Active, Grounded, In Maintenance...

**Technical Constraints**
Status data is fetched from the maintenance API...
```

**Expected Parsed Output:**
- ✅ `acs[]` = 4 entries
- ✅ ACs 3 & 4 marked `conditional: true` (IF/IF NO pattern)
- ✅ `sections[]` contains "Specified Behavior" and "Technical Constraints"
- ✅ `out_of_scope[]` = ["Editing aircraft registration data is not in scope"]
- ✅ No ad-hoc field `technical_constraints`
- ✅ `needs_review` = false

---

### FMT-003: Markdown `## Section` with Subsections

**Expected Parsed Output:**
- ✅ `acs[]` = 5 entries
- ✅ AC-1 marked `conditional: false`
- ✅ `sections[]` contains "Specified Behavior" with subsections
- ✅ Each subsection appears in `sections[N].subsections[]` with correct header and content
- ✅ `questions[]` has 2 entries; question 1 answered, question 2 unanswered
- ✅ `comments[]` has 1 entry from "Product Owner"
- ✅ `dependencies[]` populated from "Dependencies" section
- ✅ `needs_review` = false

---

### FMT-004: Informal / No Structure

**Description Format:**
```
Show a banner when there is a maintenance alert. The banner should be visible 
at the top. It should disappear when dismissed. There should be some way to see 
the full details of the alert. The banner needs to look different from the normal 
information banners. Color should probably be different. Make sure it works on 
mobile too.
```

**Expected Parsed Output:**
- ✅ `description` = entire body text (no section headers found)
- ✅ `sections: []` (empty array)
- ✅ `acs: []` (no AC section identifiable)
- ✅ `needs_review` = true
- ✅ `flags: ["NEEDS_REVIEW"]` or `["NO_ACS_FOUND"]`
- ✅ **NO invented fields** — no `requirements`, no `behavior_notes`, no `technical_details`
- ✅ Parser must pause and alert user

**CRITICAL CHECK:** No hallucinated ACs or fields created

---

### FMT-006: User Story Triple ONLY

**Expected Parsed Output:**
- ✅ `acs: []` — no ACs invented from the triple prose
- ✅ `needs_review` = true
- ✅ `flags` contains `"no_acs_found"` or `"NEEDS_REVIEW"`
- ✅ `description` = the triple sentence verbatim
- ✅ Parser must surface the gap to the user
- ✅ **NO hallucinated ACs** from "So that" clause

---

### FMT-007: ACs as "As a / I want / so that" Lines

**Expected Parsed Output:**
- ✅ `acs[]` = 5 entries (one per triple line, excluding framing triple)
- ✅ Each AC text = the full "As a ... I want ... so that ..." sentence
- ✅ All ACs marked `testable: true`
- ✅ `needs_review` = false
- ✅ Schema shape identical to FMT-001
- ✅ **Lines must not be collapsed** into single AC

---

### FMT-008: Gherkin / BDD — Given/When/Then

**Expected Parsed Output:**
- ✅ `acs[]` = 4 entries (one per Scenario block)
- ✅ Each AC text = the full scenario block (Given/When/Then preserved)
- ✅ All ACs marked `testable: true`
- ✅ `sections: []` (empty)
- ✅ `needs_review` = false
- ✅ **Scenario blocks must not be flattened**

---

### FMT-009: Numbered List Without AC Header Label

**Expected Parsed Output:**
- ✅ `acs[]` = 5 entries (one per numbered item)
- ✅ Parser infers the list is the AC section from context
- ✅ `needs_review` = false
- ✅ `sections: []` (empty)
- ✅ **No `"Acceptance Criteria"` invented in `sections[]`**
- ✅ No items dropped

---

### FMT-010: ADF JSON Description (Jira Cloud)

**Expected Parsed Output:**
- ✅ `acs[]` = 4 entries (extracted from `orderedList` content nodes)
- ✅ `out_of_scope[]` = 1 entry
- ✅ `description` = plain text from opening paragraph node
- ✅ **Raw ADF object must NOT appear** anywhere
- ✅ No ad-hoc field `adf_content` or `raw_description`
- ✅ Schema shape identical to FMT-001
- ✅ `needs_review` = false

---

### FMT-011: HTML Description (Azure DevOps / Older Jira)

**Expected Parsed Output:**
- ✅ `acs[]` = 4 entries (from `<li>` items in `<ol>`)
- ✅ `out_of_scope[]` = 1 entry
- ✅ `sections[]` = 1 entry for "Technical Constraints"
- ✅ `description` = plain text of opening `<p>` (HTML tags stripped)
- ✅ **No HTML tags in any field value** — all output is plain text
- ✅ No ad-hoc field `html_body` or `raw_html`
- ✅ `needs_review` = false

---

## Mixed Batch Test (FMT-005)

**Setup:**
1. Both FMT-001.raw.json and FMT-004.raw.json already copied
2. Both registry entries already added

**Execution:**
```
1. Load @parser
2. Say: "Parse stories TEST-FMT-001 and TEST-FMT-004"
3. Let parser complete both
```

**Expected Parsed Output:**
- ✅ Both output files exist and are valid JSON
- ✅ Both have the same top-level key set (even if values differ)
- ✅ FMT-004 output has `acs: []` and `sections: []`
- ✅ FMT-001 output has populated arrays
- ✅ Schema shape (key names) is identical across both files
- ✅ `flags[]` differ appropriately (FMT-001 empty, FMT-004 has NEEDS_REVIEW)

---

## Result Documentation Format

After each test, record:

```
### Test: FMT-00N — [Format Description]

**Fixture:** test-harness/parser-formats/FMT-00N.raw.json  
**Parsed Output:** test-harness/project-output-epics/stories/parsed/TEST-FMT-00N.parsed.json

**ASSERTION 1: [Description]**
Expected: [What should happen]
Actual: [What actually happened]
Status: ✅ PASS / ❌ FAIL

**ASSERTION 2: [Description]**
Expected: [What should happen]
Actual: [What actually happened]
Status: ✅ PASS / ❌ FAIL

[... repeat for all critical assertions ...]

**OVERALL STATUS:** ✅ PASS / ❌ FAIL  
**Hardening Required:** [If any fail]
```

---

## Hardening Protocol

If any test **FAILS**:

1. **Document the failure** in the Actual Result column
2. **Identify the root cause** in the parser logic or instructions
3. **Update the relevant parser rule** or instruction file
4. **Re-run the same test** to confirm the fix
5. **Record the hardening** in the results section below

---

## Execution Results

### FMT-001: Plain Text Headers
**Status:** ⏳ PENDING EXECUTION

### FMT-002: Bold Headers
**Status:** ⏳ PENDING EXECUTION

### FMT-003: Markdown Headers + Subsections
**Status:** ⏳ PENDING EXECUTION

### FMT-004: Informal / No Structure
**Status:** ⏳ PENDING EXECUTION

### FMT-006: User Story Triple Only
**Status:** ⏳ PENDING EXECUTION

### FMT-007: ACs as "As a / I want" Lines
**Status:** ⏳ PENDING EXECUTION

### FMT-008: Gherkin/BDD
**Status:** ⏳ PENDING EXECUTION

### FMT-009: Numbered List Without Label
**Status:** ⏳ PENDING EXECUTION

### FMT-010: ADF JSON (Jira Cloud)
**Status:** ⏳ PENDING EXECUTION

### FMT-011: HTML Description
**Status:** ⏳ PENDING EXECUTION

### FMT-005: Mixed Batch (FMT-001 + FMT-004)
**Status:** ⏳ PENDING EXECUTION (run after FMT-001 and FMT-004 complete)

---

## Next Steps

1. ✅ Environment setup complete
2. ⏳ **Manual execution required** — Load @parser and execute tests in order
3. ⏳ Document each result using the template above
4. ⏳ Identify any hardening needed
5. ⏳ Update adversarial-testing.md with final results
6. ⏳ Update hardening timeline

---

## Notes

- These tests are critical for validating parser robustness across 10+ format styles
- Each test must be fully documented before moving to the next
- Any test failure requires root cause analysis and hardening
- Final results will be merged into `docs/adversarial-testing.md` as "Category 14"
