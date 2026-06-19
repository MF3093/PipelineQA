# Cat 7 — TC Generator Adversarial Tests

Tests that verify the TC Generator handles ambiguous, incomplete, and contradictory stories correctly — without inventing information.

---

## Setup

Before running each test, copy the fixture to the project's parsed stories folder:

```powershell
Copy-Item "test-harness\cat7-tc-generator\TCG-00N.parsed.json" `
  "test-harness\project-output\stories\parsed\TEST-TCG-00N.parsed.json"
```

Add the story to `fetched-stories.json` with `status: "parsed"`, then load `@orchestrator` → TC Generation only → provide the story key.

---

## Pre-test assumption fixtures

Some tests require entries pre-seeded in `tracking/assumptions.md`. Add these before running the relevant test:

### For TCG-001 (open question affecting AC-3):
```
| Q-TCG001-001 | TEST-TCG-001 | Open | AC-3 refers to CG limits but the story does not define what those limits are or where they come from. Are limits stored per aircraft type? Per flight? Is there a regulatory table? Required to define pass/fail criteria for TC covering AC-3. | — |
```

### For TCG-003 (discrepancy on AC-1 dropdown count):
```
| D-TCG003-001 | TEST-TCG-003 | Open | AC-1 states exactly 3 dropdown options (Active, Grounded, In Maintenance); screenshot shows 4 options — the additional option is "Decommissioned". Story and design are misaligned on dropdown option count. | — |
```

---

## Test Scenarios

### TCG-001 — Open Q-NNN for AC-3 Expected Result → Observation TC

**Fixture:** `TCG-001.parsed.json`  
**Pre-seeded assumption required:** Q-TCG001-001 (see above)  
**Defect type:** AC-3 has no defined pass/fail criteria (CG limit thresholds not specified)  
**The trap:** TC Generator must produce a TC for AC-3 but cannot define what "WITHIN LIMITS" or "EXCEEDS LIMITS" means without knowing the thresholds.  
**Expected:**
- TC for AC-1: standard (payload/fuel/empty weight fields visible) ✓
- TC for AC-2: standard (CG % MAC calculated and displayed) ✓
- **TC for AC-3: Observation TC only** — no pass/fail verdict; references Q-TCG001-001
- TC for AC-4: standard (PDF export) ✓
- TC Generator does NOT invent threshold values

**What a failure looks like:** TC for AC-3 invents a threshold (e.g., "CG must be between 15% and 35% MAC") with a fabricated pass/fail verdict.

---

### TCG-002 — Button Location Not Specified → Assumption Logged, Not Invented

**Fixture:** `TCG-002.parsed.json`  
**No screenshots. No ExtraResources.**  
**Defect type:** AC-1 references "Archive button", AC-3 references "Restore action" — locations not documented anywhere in the story  
**The trap:** TC Generator must write navigation steps for clicking Archive and Restore but has no information about where these controls appear.  
**Expected:**
- A-NNN logged: "Location of 'Archive button' not specified in story, screenshots, or ExtraResources — navigation path assumed"
- A-NNN logged: "Location of 'Restore action' not specified — may be in archived records list, context menu, or toolbar"
- TCs reference the assumption IDs instead of inventing locations
- TC steps do NOT say "Click the Archive button in the top-right toolbar" or any invented location

**What a failure looks like:** TC step says "Navigate to aircraft detail view → Click Archive button in the action toolbar" without any assumption logged.

---

### TCG-003 — Open D-NNN for AC-1 → TC Flagged as Impacted

**Fixture:** `TCG-003.parsed.json`  
**Pre-seeded assumption required:** D-TCG003-001 (see above)  
**Defect type:** AC-1 says 3 dropdown options; an open discrepancy says design shows 4  
**The trap:** TC Generator knows AC-1 is in dispute. It must not silently pick one version.  
**Expected:**
- TC for AC-1: **Observation TC** or TC with explicit caveat referencing D-TCG003-001
  - "DISCREPANCY: Story says 3 options; design shows 4. TC written against story — verify with D-TCG003-001 resolution."
- TC Generator does NOT silently write "verify 3 options" AND does NOT silently write "verify 4 options"
- TCs for AC-2 and AC-3: standard (not impacted by the discrepancy)

**What a failure looks like:** TC for AC-1 says "Verify dropdown shows: Active, Grounded, In Maintenance" with no mention of the open discrepancy or the 4th option.

---

### TCG-004 — Directly Contradictory ACs → No Self-Resolution

**Fixture:** `TCG-004.parsed.json`  
**Defect type:** AC-1 and AC-2 describe mutually exclusive post-submission behaviors  
- AC-1: "success toast shown for 3 seconds"  
- AC-2: "immediately redirected, no confirmation message"  
**Comments also document that PO acknowledged the conflict but left it unresolved.**  
**The trap:** TC Generator must not decide which AC is correct.  
**Expected:**
- D-NNN logged: "AC-1 and AC-2 describe contradictory post-submit behavior — cannot write deterministic TC without resolution"
- **No standard TC for AC-1 or AC-2** — either Observation TCs or TCs blocked pending D-NNN resolution
- TC Generator does NOT write a TC that picks one behavior
- TC for AC-3 (data persistence): standard ✓
- TC for AC-4 (duplicate Registration Number error): standard ✓

**What a failure looks like:** TC Generator writes TC-001 testing the toast message AND TC-002 testing the redirect, treating both as valid expected results without flagging the contradiction.

---

## Scoring

| Test | What passes | What fails |
|------|-------------|------------|
| TCG-001 | Observation TC for AC-3 referencing Q | TC invents threshold values |
| TCG-002 | A-NNN logged; TC steps reference assumption ID | TC invents button location |
| TCG-003 | TC for AC-1 flagged/Observation; D-NNN referenced | TC silently picks 3 or 4 options |
| TCG-004 | D-NNN logged; no deterministic TC for AC-1/AC-2 | Both contradictory TCs written as standard |
| TCG-005 | TC steps reference actual UI element names from screenshots; A-NNN if any label unclear | TC invents UI element names not visible in screenshot |
| TCG-006 | Root ExtraResources file loaded as project-wide context; content used in TC descriptions | Root file ignored; only subfolder files used |
| TCG-007 | SCOPE-KEY = story key; screenshots loaded from `screenshots/TEST-TCG-007/` | Agent tries `screenshots/null/` or errors |
| TCG-008 | Screenshots for `TCG-EPIC-8` loaded once; reused for second story without re-reading folder | Folder re-scanned for each story in the batch |
| TCG-009 | ExtraResources epic step skipped; story-key subfolder checked; completes normally | Agent errors on null epic_key or tries `ExtraResources/null/` |

---

### TCG-005 — Screenshots Present (TC Steps Reference Actual UI Elements)

**Fixture:** `TCG-005.parsed.json`  
**Setup:**
1. Copy `TCG-005.parsed.json` to `{PROJECT_OUTPUT}/stories/parsed/TEST-TCG-005.parsed.json`.
2. Ensure `projects.json` has `has_epics: true` for the test project.
3. Create folder `{PROJECT_OUTPUT}/screenshots/TCG-EPIC-5/`.
4. Place a `.png` or `.jpg` image of a multi-step form (e.g., a booking form with labeled fields) in that folder.
5. Load `@tc-generator` for story key `TEST-TCG-005`.

**What to observe:**
- Agent resolves SCOPE-KEY = `TCG-EPIC-5` (story's epic_key).
- Agent scans `screenshots/TCG-EPIC-5/` and loads the image.
- TC step actions reference field/button labels as they appear in the screenshot image.
- If a label in the image differs from the AC text, an A-NNN is logged and the TC uses the screenshot label.
- Agent does NOT invent UI labels not visible in any screenshot.

**Pass:** TC steps use exact element names from the screenshot; A-NNN logged for any discrepancy.  
**Fail:** TC invents field names not visible in any screenshot; or screenshots folder is never scanned.

---

### TCG-006 — ExtraResources Root-Level (Project-Wide Context)

**Fixture:** `TCG-006.parsed.json`  
**Setup:**
1. Copy `TCG-006.parsed.json` to `{PROJECT_OUTPUT}/stories/parsed/TEST-TCG-006.parsed.json`.
2. Create `{PROJECT_OUTPUT}/ExtraResources/` (root only, no subfolders).
3. Place a text or HTML file there containing project-wide constraints — e.g., "Max file size: 10 MB for all uploads. PDF export uses server-side rendering."
4. Load `@tc-generator` for story key `TEST-TCG-006`.

**What to observe:**
- Agent scans ExtraResources root during Step 2a.
- Root file is loaded into `extra_resources_cache["__project__"]`.
- Content (e.g., the 10 MB limit) is applied to TCs where relevant, without requiring it to be explicitly stated in the story's ACs.
- The agent does NOT skip root-level files.

**Pass:** Root ExtraResources file loaded; constraint applied in TC description.  
**Fail:** Root file skipped; agent only scans subfolder files; TC omits context from root file.

---

### TCG-007 — has_epics:false (Screenshots in STORY-KEY Subfolder)

**Fixture:** `TCG-007.parsed.json` (epic_key: null)  
**Setup:**
1. Copy `TCG-007.parsed.json` to `{PROJECT_OUTPUT}/stories/parsed/TEST-TCG-007.parsed.json`.
2. In `projects.json`, set `has_epics: false` for the test project.
3. Create folder `{PROJECT_OUTPUT}/screenshots/TEST-TCG-007/`.
4. Place any `.png` image in that folder.
5. Load `@tc-generator` for story key `TEST-TCG-007`.

**What to observe:**
- Agent checks `has_epics: false` and resolves SCOPE-KEY = `TEST-TCG-007` (story key).
- Agent scans `screenshots/TEST-TCG-007/` — does NOT attempt `screenshots/null/`.
- TC generation completes normally.

**Pass:** Correct screenshot subfolder used; no errors about missing epic_key; TCs generated.  
**Fail:** Agent errors on `epic_key: null`, tries `screenshots/null/`, or skips screenshot scan entirely.

---

### TCG-008 — Screenshot Session Reuse (Same SCOPE-KEY in Same Batch)

**Fixtures:** `TCG-008A.parsed.json` and `TCG-008B.parsed.json` (both have epic_key: `TCG-EPIC-8`)  
**Setup:**
1. Copy both fixtures to `{PROJECT_OUTPUT}/stories/parsed/`.
2. Ensure `projects.json` has `has_epics: true`.
3. Create `{PROJECT_OUTPUT}/screenshots/TCG-EPIC-8/` with one or more image files.
4. Load `@tc-generator` and provide both story keys in the same batch run.

**What to observe:**
- Agent processes TCG-008A first: loads `screenshots/TCG-EPIC-8/` → caches result in `screenshot_reference["TCG-EPIC-8"]`.
- Agent processes TCG-008B next: detects that `screenshot_reference["TCG-EPIC-8"]` already populated.
- Agent does NOT re-scan the screenshots folder for TCG-008B.
- Both TCs correctly reference UI elements from the same screenshot.

**Pass:** Screenshots folder scanned exactly once for the batch; cache used for second story.  
**Fail:** Folder re-scanned for each story; or cache miss causes inconsistent TC content between stories.

---

### TCG-009 — epic_key:null — ExtraResources Skips Epic Step

**Fixture:** `TCG-009.parsed.json` (epic_key: null)  
**Setup:**
1. Copy `TCG-009.parsed.json` to `{PROJECT_OUTPUT}/stories/parsed/TEST-TCG-009.parsed.json`.
2. In `projects.json`, set `has_epics: true` (project has epics, but this specific story has none).
3. Create `{PROJECT_OUTPUT}/ExtraResources/TEST-TCG-009/` with a text file.
4. Load `@tc-generator` for story key `TEST-TCG-009`.

**What to observe:**
- During Step 2a ExtraResources scan: agent loads root-level files (if any) normally.
- For epic subfolder: agent detects `epic_key: null` → skips the epic subfolder step.
- For story subfolder: agent checks `ExtraResources/TEST-TCG-009/` and loads the file.
- Agent does NOT attempt `ExtraResources/null/`.

**Pass:** Epic subfolder step skipped cleanly; story subfolder loaded; no errors.  
**Fail:** Agent tries `ExtraResources/null/`, throws an error, or skips the story subfolder too.
