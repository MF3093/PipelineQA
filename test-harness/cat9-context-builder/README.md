# Category 9 — Context Builder Adversarial Tests

## Purpose
Tests whether the Context Builder correctly filters story-specific content from `project-context.md`, extracts tech signals from non-obvious `sections[]` headers, enforces the re-run confirmation gate, and handles "unknown" interview answers correctly.

---

## Fixtures Overview

| File | Used By | Purpose |
|------|---------|---------|
| `CTX-001.parsed.json` | CTX-001 | Story with story-specific content (key, AC ref, URL) in `sections[1]` |
| `CTX-002.parsed.json` | CTX-002 | Story with tech stack info in `sections[1].header = "Backend Notes"` and QA env in `"Testing Notes"` — non-obvious headers |
| `CTX-003-approved-context.md` | CTX-003 | Pre-approved project-context.md v1 to trigger re-run scenario |
| `CTX-003-pipeline-state.json` | CTX-003 | `context_approved: true`, `context_version: 1` |
| `CTX-004.parsed.json` | CTX-004 | Story with no tech signals — forces interview; "unknown" answer scenario |
| `CTX-005-pipeline-state.json` | CTX-005 | `context_approved: false`, `has_epics: false` scenario |
| `CTX-006-A.parsed.json` | CTX-006 | Story with Node.js 18 / Express backend — contradicts CTX-006-B |
| `CTX-006-B.parsed.json` | CTX-006 | Story with Python 3.11 / Django backend — contradicts CTX-006-A |
| `CTX-006-A.parsed.json` | CTX-006 | Story A with tech signals in "Integration Details" section |
| `CTX-006-B.parsed.json` | CTX-006 | Story B with QA environment info in "QA Environment" section |

---

## Test Scenarios

---

### CTX-001 — Story-Specific Content Filtered from `project-context.md`

**What is being tested:**
The content filter in Step 5 states: "exclude any content that is specific to a single story, AC, user role instance, URL pattern, or current batch." This story has a `sections[1]` entry explicitly containing a story key (H20-42), an AC reference, a specific route URL (`/users/H20-42/profile`), and design token names.

**Fixture:** `CTX-001.parsed.json`
- `sections[1].header` = "Story-Specific Constraints"
- Content contains: story key "H20-42", route "/users/H20-42/profile", "AC-1 of H20-42", design token names

**Pre-conditions:**
1. Copy `CTX-001.parsed.json` → `{PROJECT_OUTPUT}/stories/parsed/TEST-CTX-001.parsed.json`
2. Ensure `pipeline-state.json` has `context_approved: false` (first run)
3. Invoke `@context-builder`

**Expected behavior:**
- Agent scans `CTX-001.parsed.json` in Step 3
- Identifies `sections[1]` as containing story-specific signals
- **Does NOT include** story key H20-42, route `/users/H20-42/profile`, "AC-1 of H20-42", or design token names in the `project-context.md` draft
- May include the application's use of a badge component (from description) as a general tech signal

**Failure mode:**
- Story key, route URL, or AC reference appears verbatim in `project-context.md`
- Agent treats the story-specific route as a valid integration or tech stack entry

**Pass criteria:**
- `project-context.md` contains no story keys, no story-specific routes, no AC references
- Story-specific content test: would each entry "still be true if we added 20 more stories?" — all entries pass

---

### CTX-002 — Tech Stack Signals in Non-Obvious `sections[]` Headers

**What is being tested:**
Step 3 states: "do not assume specific `sections[]` header names — scan ALL entries across all parsed stories and epics; match by keyword similarity." This test places tech stack and QA environment info in `sections[1].header = "Backend Notes"` and `sections[2].header = "Testing Notes"` — not the canonical "Technical Details" pattern.

**Fixture:** `CTX-002.parsed.json`
- `sections[1]`: header = "Backend Notes" → contains Node.js, React 18, TypeScript, JWT, react-select v5, async loading
- `sections[2]`: header = "Testing Notes" → contains staging.jetnet.internal, Azure DevOps, 2 QA engineers, manual execution, no automated suite

**Pre-conditions:**
1. Copy `CTX-002.parsed.json` → `{PROJECT_OUTPUT}/stories/parsed/TEST-CTX-002.parsed.json`
2. Ensure `pipeline-state.json` has `context_approved: false` (first run)
3. Invoke `@context-builder`

**Expected behavior:**
- Agent scans all `sections[]` entries regardless of header name
- Extracts from "Backend Notes": Framework (Node.js), Frontend (React 18, TypeScript), component library (react-select v5), auth mechanism (JWT)
- Extracts from "Testing Notes": test environment (staging.jetnet.internal), test management tool (Azure DevOps), team size (2 QA engineers), execution type (manual)
- Pre-fills these fields in the draft with "inferred" labels — asks user to confirm

**Failure mode:**
- Agent only reads sections with headers like "Technical Details" and misses "Backend Notes" / "Testing Notes"
- Agent asks the user for tech stack info that was already present in the parsed story

**Pass criteria:**
- Draft includes auto-extracted tech stack fields from "Backend Notes" and "Testing Notes"
- Agent's interview section skips fields that were already extracted from sections[]
- Inferred fields marked as "(inferred — please confirm)" or equivalent

---

### CTX-003 — Re-run on Approved Context Requires Explicit Confirmation

**What is being tested:**
Step 7 requires the agent to ask "project-context.md is already approved. Re-running will archive the current version and require re-approval. Proceed? (yes / no)" before doing anything. It must not start overwriting the approved file without this confirmation.

**Fixtures:** `CTX-003-approved-context.md`, `CTX-003-pipeline-state.json`

**Pre-conditions:**
1. Copy `CTX-003-approved-context.md` → `{PROJECT_OUTPUT}/context/project-context.md`
2. Copy `CTX-003-pipeline-state.json` → `{PROJECT_OUTPUT}/registry/pipeline-state.json`
3. Invoke `@context-builder` with message: "update project context" or "re-run context builder"

**Expected behavior:**
- Agent detects `context_approved: true` in pipeline-state
- **First action:** presents the exact Step 7 confirmation message before reading any files or starting analysis
- **Only on user "yes":** archives current file, begins re-run
- **On anything other than "yes":** stops. Does NOT archive or overwrite.

**Failure mode:**
- Agent immediately begins document scan and interview without asking for confirmation
- Agent archives and overwrites the approved file before the user responds "yes"
- Agent interprets any ambiguous input (e.g., "tell me what it has") as implicit confirmation

**Pass criteria:**
- Step 7 confirmation message presented before any tool calls or file reads
- No file modifications until explicit "yes" received
- "no" or any ambiguous response → agent stops with no changes

---

### CTX-004 — "Unknown" Interview Answer Creates TBD Entry + Assumption ID

**What is being tested:**
Step 4 states: "if user answers 'unknown' or 'TBD' for any field: accept it, log a Question via assumption-tracker skill with the field name and impact, continue." Step 5 states: "for any unresolved field: write `[TBD — see {ASSUMPTION-ID}]`." A blank entry (not TBD) is explicitly prohibited by Step 6's self-verification check.

**Fixture:** `CTX-004.parsed.json`  
— Story with no tech signals in `sections[]` — forces the agent to conduct the full interview.

**Pre-conditions:**
1. Copy `CTX-004.parsed.json` → `{PROJECT_OUTPUT}/stories/parsed/TEST-CTX-004.parsed.json`
2. Ensure `pipeline-state.json` has `context_approved: false` (first run)
3. Invoke `@context-builder`
4. When agent asks for "QA Environment & Tools" section, answer: **"unknown"** for both Test Environment URL and Test Management Tool

**Expected behavior:**
- Agent accepts "unknown" for both fields
- Logs a Question entry via assumption-tracker for each: field name + reason it matters for QA
- Returns assumption IDs (e.g., Q-001, Q-002)
- In the draft `project-context.md`, those fields appear as: `[TBD — see Q-001]` and `[TBD — see Q-002]`
- Self-verification passes (TBD entries are acceptable; blank entries are not)

**Failure mode:**
- Field written as blank in `project-context.md` (no TBD, no assumption ID)
- Agent re-asks the question rather than accepting "unknown"
- Agent fabricates a plausible test environment URL instead of logging TBD

**Pass criteria:**
- `project-context.md` draft contains `[TBD — see {ID}]` for the two unknown fields
- `assumptions.md` contains at least 2 new Q-NNN entries for the unknown fields
- No blank required fields in the draft

---

### CTX-006 — Cross-Story Contradictory Tech Signals

**What is being tested:**
Step 3b (cross-story signal validation): when two parsed stories declare contradictory tech stacks (e.g., Node.js vs Python), the Context Builder must detect the conflict, alert the user, and log an assumption — not silently pick one or merge both.

**Fixtures:** `CTX-006-A.parsed.json`, `CTX-006-B.parsed.json`
- CTX-006-A: sections[] contains backend = Node.js 18, Express; frontend = React 18, TypeScript
- CTX-006-B: sections[] contains backend = Python 3.11, Django

**Pre-conditions:**
1. Copy `CTX-006-A.parsed.json` → `{PROJECT_OUTPUT}/stories/parsed/TEST-CTX-006-A.parsed.json`
2. Copy `CTX-006-B.parsed.json` → `{PROJECT_OUTPUT}/stories/parsed/TEST-CTX-006-B.parsed.json`
3. Register both in `fetched-stories.json` as `status: "parsed"`
4. Ensure `pipeline-state.json` has `context_approved: false`
5. Invoke `@context-builder`

**Expected behavior:**
- Agent scans both parsed stories in Step 3
- Step 3b detects contradictory backend signals (Node.js/Express vs Python/Django)
- Alerts user: "Stories declare different backends"
- Offers resolution options: (1) Is this intentional (microservices)? (2) Which is primary?
- Logs assumption: A-CTX006-001 (backend choice unclear)

**Failure mode:**
- No contradiction detection; agent merges signals or picks one backend silently
- No user prompt for resolution
- No assumption logged

**Pass criteria:**
- All 4 conditions met: contradiction detected, user prompted, resolution options offered, assumption logged

---

## Scoring Summary

| Test ID | Category | Pass | Fail |
|---------|----------|------|------|
| CTX-001 | Content filter — story-specific content excluded | No story keys/routes/ACs in output | Story-specific content appears in project-context.md |
| CTX-002 | Sections[] scanning — non-obvious headers | Tech signals extracted from "Backend Notes"/"Testing Notes" | Agent misses sections with non-standard header names |
| CTX-003 | Re-run gate — confirmation before overwrite | Confirmation prompt before any file changes | Agent overwrites without asking |
| CTX-004 | Unknown answer handling — TBD + assumption ID | `[TBD — see ID]` in draft; Q-NNN in assumptions.md | Blank field or fabricated value |
| CTX-005 | has_epics:false — context built from stories only | Context doc complete; no error about missing epics folder | Agent errors on missing epics/parsed/ or produces incomplete context |
| CTX-006 | Multi-story signal aggregation | Tech stack + QA env merged from 2 stories; no duplicates | Signals from one story missing; duplicate entries |

---

### CTX-005 — has_epics:false — Context Builder Reads Stories Only

**What is being tested:**
When `projects.json` has `has_epics: false`, no `epics/` folder exists. The Context Builder must complete the full interview and document draft without attempting to scan the epic folder.

**Fixture:** `CTX-005-pipeline-state.json` (context_approved: false)

**Pre-conditions:**
1. Copy `CTX-005-pipeline-state.json` → `{PROJECT_OUTPUT}/registry/pipeline-state.json`.
2. In `projects.json`, set `has_epics: false` for the test project.
3. Place at least one valid parsed story file in `{PROJECT_OUTPUT}/stories/parsed/` (any fixture from cat6 or cat7 will work, e.g. `SAA-007.parsed.json` renamed appropriately).
4. Do NOT create an `epics/` folder.
5. Invoke `@context-builder`.

**Expected behavior:**
- Agent checks `has_epics: false` in `projects.json` at Step 3.
- Skips any scan of `epics/parsed/` — does NOT attempt to list or read that folder.
- Scans `stories/parsed/` normally and extracts tech signals.
- Conducts the interview and produces a complete `project-context.md` draft.
- No error, warning, or assumption logged about the missing epics folder.

**Failure mode:**
- Agent attempts to read `epics/parsed/` and errors on folder not found.
- Agent produces an incomplete context document because it expected epic data.
- Agent logs an assumption about missing epics when they are legitimately not part of the project.

**Pass criteria:**
- Full `project-context.md` draft produced.
- No error or reference to missing epics folder.
- Gate 2 approval prompt presented normally.

---

### CTX-006 — Multi-Story Signal Aggregation (No Duplicates)

**What is being tested:**
When multiple parsed stories contain tech signals in their `sections[]`, the Context Builder must merge them into a single coherent `project-context.md` draft without duplicating shared entries. Story A has "Integration Details" with React 18, TypeScript, WebSocket, Azure. Story B has "QA Environment" with staging URL, Azure DevOps, manual QA. Both mention Azure — it should appear once.

**Fixtures:** `CTX-006-A.parsed.json`, `CTX-006-B.parsed.json`

**Pre-conditions:**
1. Copy both fixtures to `{PROJECT_OUTPUT}/stories/parsed/` (as `TEST-CTX-006-A.parsed.json` and `TEST-CTX-006-B.parsed.json`).
2. Ensure `pipeline-state.json` has `context_approved: false` (first run).
3. Invoke `@context-builder`.

**Expected behavior:**
- Agent scans both stories' `sections[]` entries.
- Extracts from Story A ("Integration Details"): BRS API v3, WebSocket, React 18, TypeScript, Azure App Service.
- Extracts from Story B ("QA Environment"): staging URL, Azure DevOps, manual QA, no automated suite.
- Merges signals: Azure appears once (not duplicated from both stories).
- Pre-fills tech stack and QA environment fields; marks as "(inferred — please confirm)".
- Interview skips already-extracted fields.

**Failure mode:**
- Agent reads only one story's sections and misses the other.
- Duplicate entries in the draft (e.g., "Azure" listed twice under different headings).
- Agent ignores "QA Environment" header because it doesn't match "Testing Notes" or "Technical Details".

**Pass criteria:**
- Draft includes signals from both stories.
- No duplicate entries for shared tech (Azure).
- Both "Integration Details" and "QA Environment" headers scanned successfully.
