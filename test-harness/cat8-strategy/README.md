# Category 8 — Story Prioritizer agent Adversarial Tests

## Purpose
These fixtures test the Story Prioritizer agent's scoring integrity, extension mode discipline, state write accuracy, comment-signal discrimination, and prerequisite gate enforcement. All tests target behaviors explicitly defined in `story-prioritizer.agent.md` Step 1–5 and the scoring model.

---

## Fixtures Overview

| File | Used By | Purpose |
|------|---------|---------|
| `project-context.fixture.md` | All tests | Shared project context — copy to `{PROJECT_OUTPUT}/context/project-context.md` |
| `STR-001.parsed.json` | STR-001 | Design reference buried in `sections[]`, `figma_links` is empty |
| `STR-002A.parsed.json` | STR-002 | Batch-001 story — already in approved matrix, DW=0 |
| `STR-002B.parsed.json` | STR-002 | Batch-002 story — depends on STR-002A |
| `STR-002-pipeline-state.json` | STR-002 | Pre-set state: strategy approved, only 002A scoped |
| `STR-002-approved-matrix.md` | STR-002 | v1 matrix with TEST-STR-002A, Dependency Weight = 0 |
| `STR-003A–C.parsed.json` | STR-003 | Batch-001 stories — already in approved matrix |
| `STR-003D–E.parsed.json` | STR-003 | Batch-002 stories — both depend on TEST-STR-003C |
| `STR-003-pipeline-state.json` | STR-003 | Pre-set state: strategy approved, only A–C scoped |
| `STR-003-approved-matrix.md` | STR-003 | v1 matrix with 003A, 003B, 003C — all DW=0 |
| `STR-004.parsed.json` | STR-004 | Clear ACs + 6 administrative-noise comments |
| `STR-005-pipeline-state.json` | STR-005 | `context_approved: false` — prerequisite gate not met |

---

## Test Scenarios

---

### STR-001 — Severity Amplifier: Design Reference in `sections[]`, `figma_links` Empty

**What is being tested:**
The severity amplifier rule states: scan `sections[]` for any entry whose header or content contains a Figma URL (or other design tool URL / the words "design", "mockup", "prototype"). The amplifier is applied only if NO such reference exists anywhere in the story (including `description`). This test checks whether the agent correctly finds the URL in `sections[1].content` even when `figma_links` is empty.

**Fixture:** `STR-001.parsed.json`
- `figma_links: []` — empty (no shortcut)
- `sections[1].content` — contains a `https://www.figma.com/file/...` URL

**Pre-conditions:**
1. Copy `project-context.fixture.md` → `{PROJECT_OUTPUT}/context/project-context.md`
2. Copy `STR-001.parsed.json` → `{PROJECT_OUTPUT}/stories/parsed/TEST-STR-001.parsed.json`
3. Ensure `pipeline-state.json` has `context_approved: true` and no existing `priority-matrix.md`
4. Invoke `@story-prioritizer` — provide TEST-STR-001 as the batch

**Expected behavior:**
- Agent scans all of `sections[]` for design tool references
- Finds the Figma URL in `sections[1].content`
- Does **NOT** apply the severity amplifier (+1) to this story
- Severity reflects the story's actual functional risk without inflation

**Failure mode:**
- Agent only checks `figma_links[]` → finds it empty → applies +1 amplifier → inflated severity score

**Pass criteria:**
- Severity amplifier NOT applied to TEST-STR-001
- Row in priority matrix reflects Severity without +1
- No assumption logged about missing design reference for this story

---

### STR-002 — Extension Mode: Only Dependency Weight Updated on Approved Row

**What is being tested:**
In extension mode, only `Dependency Weight` and `Functional Role` may be updated on existing approved rows. All other columns (Severity, Likelihood, Risk Description, Rank) are locked. This test introduces a new story (002B) that depends on an existing approved story (002A), forcing a DW update on 002A from 0 → 1.

**Fixtures:** `STR-002A.parsed.json`, `STR-002B.parsed.json`, `STR-002-approved-matrix.md`, `STR-002-pipeline-state.json`

**Pre-conditions:**
1. Copy `project-context.fixture.md` → `{PROJECT_OUTPUT}/context/project-context.md`
2. Copy `STR-002A.parsed.json` → `{PROJECT_OUTPUT}/stories/parsed/TEST-STR-002A.parsed.json`
3. Copy `STR-002B.parsed.json` → `{PROJECT_OUTPUT}/stories/parsed/TEST-STR-002B.parsed.json`
4. Copy `STR-002-approved-matrix.md` → `{PROJECT_OUTPUT}/strategy/priority-matrix.md`
5. Copy `STR-002-pipeline-state.json` → `{PROJECT_OUTPUT}/registry/pipeline-state.json`
6. Create `{PROJECT_OUTPUT}/strategy/strategy-versions/` folder (empty — agent will archive into it)
7. Invoke `@story-prioritizer` — provide TEST-STR-002B as the new batch

**Expected behavior:**
- Agent detects extension mode (priority-matrix.md exists)
- Archives v1 to `strategy-versions/priority-matrix-v1.md`
- Re-evaluates Dependency Weight for TEST-STR-002A: 0 → 1 (002B depends on it)
- Score for TEST-STR-002A updates: (3×1)+1 = 4
- **Only DW and Score columns change** on the 002A row — all other columns identical to v1
- Delta view clearly shows the DW change before approval gate
- New row added for TEST-STR-002B

**Failure mode:**
- Agent modifies Severity, Likelihood, or Risk Description on the 002A row (locked columns mutated)
- Agent fails to re-evaluate DW at all (misses the dependency signal)

**Pass criteria:**
- 002A row: only `Dependency Weight` and `Score` differ from v1
- 002A row: `Severity`, `Likelihood`, `Risk Description`, `Functional Role` unchanged
- Delta view shows DW update explicitly

---

### STR-003 — `scoped_story_keys` Completeness After Extension Approval

**What is being tested:**
When Step 5 approves an extension run, the agent writes `scoped_story_keys` to `pipeline-state.json`. This must include ALL keys from ALL batches (batch-001 + batch-002), not just the keys added in the current batch. This test verifies that batch-001 keys are merged, not overwritten.

**Fixtures:** `STR-003A–C.parsed.json` (batch-001, in matrix), `STR-003D–E.parsed.json` (batch-002, new), `STR-003-approved-matrix.md`, `STR-003-pipeline-state.json`

**Pre-conditions:**
1. Copy `project-context.fixture.md` → `{PROJECT_OUTPUT}/context/project-context.md`
2. Copy STR-003A, STR-003B, STR-003C → `{PROJECT_OUTPUT}/stories/parsed/TEST-STR-003{A,B,C}.parsed.json`
3. Copy STR-003D, STR-003E → `{PROJECT_OUTPUT}/stories/parsed/TEST-STR-003{D,E}.parsed.json`
4. Copy `STR-003-approved-matrix.md` → `{PROJECT_OUTPUT}/strategy/priority-matrix.md`
5. Copy `STR-003-pipeline-state.json` → `{PROJECT_OUTPUT}/registry/pipeline-state.json`
6. Create `{PROJECT_OUTPUT}/strategy/strategy-versions/` folder (empty)
7. Invoke `@story-prioritizer` — provide TEST-STR-003D and TEST-STR-003E as the new batch

**Expected behavior after approval:**
- `pipeline-state.json.scoped_story_keys` = `["TEST-STR-003A", "TEST-STR-003B", "TEST-STR-003C", "TEST-STR-003D", "TEST-STR-003E"]` (all 5)
- `strategy_current_version` = 2
- TEST-STR-003C DW updated from 0 → 2 (both D and E depend on it); Score updated to 5

**Failure mode:**
- `scoped_story_keys` written as only `["TEST-STR-003D", "TEST-STR-003E"]` — batch-001 keys lost
- Downstream: orchestrator can no longer route 003A, 003B, 003C for TC generation

**Pass criteria:**
- All 5 keys present in `scoped_story_keys` after approval
- Order may vary; completeness is the check
- TEST-STR-003C DW updated to 2; no other existing row columns mutated

---

### STR-004 — Administrative Comment Noise Does Not Inflate Likelihood

**What is being tested:**
The comment processing rule states: act only when a comment "changes the risk picture compared to the ACs alone." Pure administrative comments (sprint assignments, PR links, status changes) must not inflate Likelihood. This story has unambiguous, well-defined ACs and 6 purely administrative comments.

**Fixture:** `STR-004.parsed.json`
- ACs are clear and complete (file browser, PDF-only validation, error message, filename display)
- `comments[]` contains: sprint move, QA assignment, point estimate, PR merge notice, status change, sprint start note — all administrative, none introducing ambiguity

**Pre-conditions:**
1. Copy `project-context.fixture.md` → `{PROJECT_OUTPUT}/context/project-context.md`
2. Copy `STR-004.parsed.json` → `{PROJECT_OUTPUT}/stories/parsed/TEST-STR-004.parsed.json`
3. Ensure `pipeline-state.json` has `context_approved: true` and no existing `priority-matrix.md`
4. Invoke `@story-prioritizer` — provide TEST-STR-004 as the batch

**Expected behavior:**
- Agent reads all 6 comments
- Determines no comment changes the risk picture: no conflicting interpretation, no unresolved debate, no new constraint
- **Likelihood = 1** (Low) — story is straightforward, ACs are unambiguous
- No assumption logged from comment content

**Failure mode:**
- Agent raises Likelihood to 2 or 3 citing sprint history or PR merge as an ambiguity signal
- Agent logs an unnecessary assumption for a comment like "PR #512 merged"

**Pass criteria:**
- `Likelihood` = 1 in the priority matrix row
- No Q-NNN or A-NNN logged that originates from these administrative comments

---

### STR-005 — Prerequisite Gate: `context_approved: false`

**What is being tested:**
Step 1 requires `pipeline-state.json` to show `context_approved: true` before the strategy may proceed. This is the same prerequisite gate class that was fixed for BYP-001. The test verifies the agent halts immediately and does not produce any partial output.

**Fixture:** `STR-005-pipeline-state.json` — `context_approved: false`, `strategy_approved: false`

**Pre-conditions:**
1. Copy `project-context.fixture.md` → `{PROJECT_OUTPUT}/context/project-context.md`
2. Copy ANY parsed story file → `{PROJECT_OUTPUT}/stories/parsed/` (content irrelevant)
3. Copy `STR-005-pipeline-state.json` → `{PROJECT_OUTPUT}/registry/pipeline-state.json`
4. Invoke `@story-prioritizer`

**Expected behavior:**
- Agent reads `pipeline-state.json` at Step 1
- Finds `context_approved: false`
- Halts immediately with the exact message defined in Step 1:
  `"project-context.md must be approved before the strategy can be generated."`
- No analysis performed, no matrix file created, no assumptions logged

**Failure mode:**
- Agent proceeds past Step 1 despite `context_approved: false`
- Agent produces partial or full strategy output
- Agent reads parsed story files despite the failed prerequisite check

**Pass criteria:**
- Agent halts at Step 1 with the required message
- No `priority-matrix.md` file created
- No parsed story files read

---

## Scoring Summary

| Test ID | Category | Pass | Fail |
|---------|----------|------|------|
| STR-001 | Scoring integrity — design reference detection | Amplifier not applied | Amplifier applied to story with Figma in sections[] |
| STR-002 | Extension mode discipline — locked row mutation | Only DW/Score updated | Any other column mutated on existing row |
| STR-003 | State write accuracy — scoped_story_keys merge | All 5 keys present | Only batch-002 keys written |
| STR-004 | Comment discrimination — noise vs. signal | Likelihood=1, no phantom assumptions | Likelihood inflated by administrative comment |
| STR-005 | Prerequisite gate enforcement | Halt at Step 1 | Any progress past the check |
| STR-006 | epic_key:null — story scored from content only | Score produced; no epic file read | Agent errors on missing epic or skips story |
| STR-007 | has_epics:false project — no epics folder | Score produced; no epic lookup attempted | Agent errors on missing epics/parsed/ folder |

---

### STR-006 — epic_key:null — Prioritizer Scores from Story Content Only

**What is being tested:**
When `epic_key` is null, the prioritizer must score the story from its own content (description, ACs, sections, comments) without attempting to read any epic file. No error must be raised.

**Fixture:** `STR-006.parsed.json` (epic_key: null)

**Pre-conditions:**
1. Copy `project-context.fixture.md` → `{PROJECT_OUTPUT}/context/project-context.md`.
2. Copy `STR-006.parsed.json` → `{PROJECT_OUTPUT}/stories/parsed/TEST-STR-006.parsed.json`.
3. Ensure `pipeline-state.json` has `context_approved: true` and no existing `priority-matrix.md`.
4. Do NOT create any file in `epics/parsed/` for this story.
5. Invoke `@story-prioritizer` — provide `TEST-STR-006` as the batch.

**Expected behavior:**
- Agent detects `epic_key: null`.
- Skips any step that would read an epic file for context.
- Scores the story based on its own ACs, description, and sections only.
- Produces a valid priority matrix row for TEST-STR-006.
- No error, warning, or assumption logged about the missing epic.

**Failure mode:**
- Agent attempts to read `epics/parsed/null.parsed.json` and errors.
- Agent skips the story silently without producing a row.
- Agent logs a phantom assumption about a missing epic.

**Pass criteria:**
- Valid matrix row produced for TEST-STR-006.
- No epic file read or referenced.
- No error raised.

---

### STR-007 — has_epics:false Project — Prioritizer Completes Without Epic Folder

**What is being tested:**
When `projects.json` has `has_epics: false` for the active project, no `epics/` folder exists. The prioritizer must complete a full first-time matrix for a batch of stories without attempting to access the missing folder.

**Fixtures:** `STR-007A.parsed.json`, `STR-007B.parsed.json`, `STR-007-pipeline-state.json`

**Pre-conditions:**
1. Copy `project-context.fixture.md` → `{PROJECT_OUTPUT}/context/project-context.md`.
2. Copy `STR-007A.parsed.json` → `{PROJECT_OUTPUT}/stories/parsed/TEST-STR-007A.parsed.json`.
3. Copy `STR-007B.parsed.json` → `{PROJECT_OUTPUT}/stories/parsed/TEST-STR-007B.parsed.json`.
4. Copy `STR-007-pipeline-state.json` → `{PROJECT_OUTPUT}/registry/pipeline-state.json`.
5. In `projects.json`, set `has_epics: false` for the test project.
6. Do NOT create an `epics/` folder.
7. Invoke `@story-prioritizer` — provide `TEST-STR-007A` and `TEST-STR-007B` as the batch.

**Expected behavior:**
- Agent checks `has_epics: false` in `projects.json`.
- Proceeds without reading or listing the `epics/` folder at any step.
- Scores both stories from their own content.
- Produces a valid priority matrix with both rows.
- DW for TEST-STR-007B set to 1 (depends on TEST-STR-007A).

**Failure mode:**
- Agent attempts to access `epics/parsed/` and throws a file-not-found error.
- Agent scores stories as 0 because it could not read epic context.

**Pass criteria:**
- Two valid rows in the priority matrix.
- No error or assumption related to missing epic folder.
- TEST-STR-007B Dependency Weight = 1.
