# Cat 6 — Story Analyzer Adversarial Tests

Tests that verify the Story Analyzer correctly surfaces internal story defects without screenshots.

---

## Setup

Before running each test, copy the fixture to the project's parsed stories folder:

```powershell
Copy-Item "test-harness\cat6-story-analyzer\SAA-00N.parsed.json" `
  "test-harness\project-output\stories\parsed\TEST-SAA-00N.parsed.json"
```

Then load `@story-analyzer` and provide the story key.

---

## Test Scenarios

### SAA-001 — Internal Contradiction: Mandatory Fields vs. Optional Description

**Fixture:** `SAA-001.parsed.json`  
**Defect type:** Internal contradiction between description and ACs  
**The trap:** Description says *"All fields are optional for draft submissions"* but AC-1 and AC-2 say Registration Number and Aircraft Type are **mandatory** before saving a draft.  
**Expected:**
- D-NNN logged: "AC-1/AC-2 state fields are mandatory; description states all fields are optional for drafts"
- Analyzer does NOT silently accept one side as correct
- Analyzer does NOT invent a resolution
- Presented to user with pause

**What a failure looks like:** Analyzer ignores the contradiction and proceeds to gap analysis without flagging it.

---

### SAA-002 — All ACs Are Vague (No Measurable Criteria)

**Fixture:** `SAA-002.parsed.json`  
**Defect type:** AC_QUALITY_ISSUE — 4 vague ACs, none testable  
**The trap:** Every AC uses unmeasurable language: "works correctly", "as expected", "properly notified", "functions as expected"  
**Expected:**
- Q-NNN logged for each vague AC asking for specific, measurable criteria
- Analyzer does NOT attempt to invent what "works correctly" means
- At minimum 4 Q-NNN entries created (one per AC)
- `needs_review: true` already set — analyzer should acknowledge but not skip

**What a failure looks like:** Analyzer treats vague ACs as valid and proceeds to produce an analysis summary without flagging them.

---

### SAA-003 — Happy Path Only (Validation Rules With No Error Behaviors)

**Fixture:** `SAA-003.parsed.json`  
**Defect type:** Missing error/failure scenarios for each validation rule  
**The trap:** 3 validation rules defined (AC-1: format, AC-2: date, AC-3: dropdown) but no story field describes what happens when each validation fails — no error messages, no field highlighting, no recovery path.  
**Expected:**
- Q-NNN for AC-1: "What error message/behavior when Flight ID format is invalid?"
- Q-NNN for AC-2: "What error message/behavior when Departure Date is in the past?"
- Q-NNN for AC-3: "What happens if no aircraft is selected? Is Submit disabled or does an error appear?"
- Analyzer does NOT invent error messages

**What a failure looks like:** Analyzer only notes the happy path ACs and produces no questions about missing error behaviors.

---

### SAA-004 — Open Questions Already Logged (No Re-logging)

**Fixture:** `SAA-004.parsed.json`  
**Defect type:** Analyzer must detect existing Q-NNN entries and NOT duplicate them  
**Pre-test setup required:**
1. Add these entries to `test-harness/project-output/tracking/assumptions.md` before running:
```
| Q-SAA004-001 | TEST-SAA-004 | Open | What is the source of the fuel capacity data — pulled from fuel management API or stored locally? | — |
| Q-SAA004-002 | TEST-SAA-004 | Open | Should fuel capacity display in liters, gallons, or be configurable per user preference? | — |
```

**The trap:** Story has `questions[]` containing those 2 items AND comments mention a blocked dependency on FUEL-API-001 — a NEW gap the analyzer should find.  
**Expected:**
- Analyzer does NOT re-log Q-SAA004-001 or Q-SAA004-002
- Analyzer DOES log a NEW Q-NNN: "Story depends on FUEL-API-001 which is unresolved per comments — what is the fallback behavior if the API is unavailable?"
- Net result: 0 duplicate entries, 1+ new entries

**What a failure looks like:** Analyzer re-logs Q-SAA004-001 and Q-SAA004-002 as new entries, creating duplicates.

---

### SAA-005 — AC Contradicts out_of_scope

**Fixture:** `SAA-005.parsed.json`  
**Defect type:** AC-3 directly contradicts `out_of_scope[]`  
**The trap:** `out_of_scope` says *"Error handling and error messages are out of scope"* but AC-3 says *"An error message is displayed if the status update fails due to a server error."*  
**Expected:**
- D-NNN logged: "AC-3 defines error message behavior; out_of_scope explicitly excludes error handling and error messages — direct contradiction"
- Analyzer does NOT silently adopt either side
- Analyzer pauses for user confirmation before proceeding

**What a failure looks like:** Analyzer skips the out_of_scope contradiction and treats AC-3 as valid without flagging it.

---

## Scoring

| Test | What passes | What fails |
|------|-------------|------------|
| SAA-001 | D-NNN logged for mandatory vs. optional contradiction | Contradiction not detected |
| SAA-002 | Q-NNN per vague AC | Vague ACs treated as testable |
| SAA-003 | Q-NNN per missing error behavior | No questions about error paths |
| SAA-004 | 0 duplicates + new gap found | Existing questions re-logged |
| SAA-005 | D-NNN for AC-vs-out_of_scope contradiction | Contradiction not detected |
| SAA-006 | Visual discrepancy step runs; screenshot loaded from EPIC-KEY subfolder | Agent skips screenshot scan or uses wrong folder path |
| SAA-007 | ExtraResources root file loaded into `extra_resources_ref["__project__"]` | Root file ignored; only subfolder files loaded |
| SAA-008 | SCOPE-KEY = STORY-KEY; screenshots loaded from `screenshots/TEST-SAA-008/` | Agent tries `screenshots/null/` or errors on missing epic_key |

---

### SAA-006 — Screenshots Present (EPIC-KEY Subfolder)

**Fixture:** `SAA-006.parsed.json`  
**Setup:**
1. Copy `SAA-006.parsed.json` to `{PROJECT_OUTPUT}/stories/parsed/TEST-SAA-006.parsed.json`.
2. Create folder `{PROJECT_OUTPUT}/screenshots/SAA-EPIC-6/`.
3. Place any `.png` or `.jpg` file in that folder (a UI mockup or any image with visible elements).
4. Ensure `projects.json` has `has_epics: true` for the test project.
5. Load `@story-analyzer` for story key `TEST-SAA-006`.

**What to observe:**
- Agent resolves SCOPE-KEY as `SAA-EPIC-6` (the story's `epic_key`).
- Agent scans `screenshots/SAA-EPIC-6/` for image files.
- Step 4A (visual discrepancy detection) executes — agent lists UI elements visible in the image.
- If AC mentions a UI element that is absent from the screenshot (or vice versa), a D-NNN is logged.

**Pass:** Agent loads the correct screenshot subfolder, runs visual analysis, documents findings.  
**Fail:** Agent skips the screenshot step, loads from wrong path, or crashes on folder lookup.

---

### SAA-007 — ExtraResources Root-Level Project-Wide File

**Fixture:** `SAA-007.parsed.json`  
**Setup:**
1. Copy `SAA-007.parsed.json` to `{PROJECT_OUTPUT}/stories/parsed/TEST-SAA-007.parsed.json`.
2. Create folder `{PROJECT_OUTPUT}/ExtraResources/` (no subfolder).
3. Place a plain text or HTML file in the root of `ExtraResources/` — e.g., `glossary.html` containing a list of system-wide terms and their definitions.
4. Load `@story-analyzer` for story key `TEST-SAA-007`.

**What to observe:**
- Agent scans `ExtraResources/` and finds the root-level file.
- The file is loaded and stored under `extra_resources_ref["__project__"]`.
- Content from this file may inform the analysis (e.g., a term defined in the glossary used in an AC is resolved using the definition from the root file).
- The agent does NOT attempt to look for the file in an epic or story subfolder.

**Pass:** Root file loaded into `extra_resources_ref["__project__"]`; project-wide context applied to analysis.  
**Fail:** Root file ignored; agent only scans subfolders; or `extra_resources_ref["__project__"]` is never populated.

---

### SAA-008 — has_epics:false (Screenshots in STORY-KEY Subfolder)

**Fixture:** `SAA-008.parsed.json` (epic_key: null)  
**Setup:**
1. Copy `SAA-008.parsed.json` to `{PROJECT_OUTPUT}/stories/parsed/TEST-SAA-008.parsed.json`.
2. In `projects.json`, set `has_epics: false` for the test project.
3. Create folder `{PROJECT_OUTPUT}/screenshots/TEST-SAA-008/`.
4. Place any `.png` or `.jpg` image file in that folder.
5. Load `@story-analyzer` for story key `TEST-SAA-008`.

**What to observe:**
- Agent checks `has_epics` in `projects.json` and gets `false`.
- Agent resolves SCOPE-KEY = `TEST-SAA-008` (story key, since epic_key is null and has_epics is false).
- Agent loads screenshots from `screenshots/TEST-SAA-008/` (not from `screenshots/null/` or any epic path).
- Visual analysis runs normally.

**Pass:** SCOPE-KEY resolves to story key; correct subfolder loaded; no errors about missing epic.  
**Fail:** Agent attempts `screenshots/null/`, throws an error, skips the screenshot step, or uses wrong path.
