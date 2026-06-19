# Category 5 — Orchestrator: Project Registration & Run Startup

Tests the Orchestrator's project registration flow, optional folder provisioning, run menu enforcement, routing logic, and startup integrity checks.
No live Jira connection required for any scenario in this category.

---

## Fixtures Overview

| File | Used By | Purpose |
|------|---------|---------|
| `ORC-003-projects.fixture.json` | ORC-003 | Pre-registered project entry for projects.json || `ORC-007-projects.fixture.json` | ORC-007 | Three registered projects for invalid selection test |
| `ORC-008-pipeline-state.fixture.json` | ORC-008 | State with strategy approved before context re-approval |
| `ORC-012-projects.fixture.json` | ORC-012 | Project with non-existent output path |
| `ORC-013-pipeline-state.fixture.json` | ORC-013 | State with context_approved = true, strategy_approved = true |
---

## Test Scenarios

---

### ORC-001 — Register New Project: All Optional Features Enabled

**What is being tested:**
Registration flow with `has_epics: yes`, `has_screenshots: yes`, `has_extra_resources: yes`.
Verifies all conditional folders and registry files are created.

**Pre-conditions:**
- `projects.json` does not contain the test project (or delete it before running)
- Choose a temp output path that does not already exist

**Steps:**
1. Load `@orchestrator` and type: `Start a new project`
2. Answer the registration questions:
   - Project name: `TEST-ORC-001`
   - Output directory: `{TEMP_PATH}/TEST-ORC-001`
   - Project key: `ORC`
   - Stories grouped under epics? → `yes`
   - UI screenshots available? → `yes`
   - Supplementary documentation? → `yes`

**Expected behavior:**
- Orchestrator creates all folders:
  `registry/`, `stories/raw/`, `stories/parsed/`, `epics/raw/`, `epics/parsed/`,
  `context/`, `strategy/strategy-versions/`, `test-cases/`, `tracking/archive/`,
  `tracking/reviews/`, `screenshots/`, `ExtraResources/`, `config/`
- Initializes: `fetched-stories.json`, `fetched-epics.json`, `pipeline-state.json`, `corrections-log.md`
- Copies `source-config.md` template to `config/source-config.md`
- Reports project registered and prompts user to fill in source-config.md
- `projects.json` entry includes: `has_epics: true`, `has_screenshots: true`, `has_extra_resources: true`

**Failure modes:**
- Optional folders (`epics/`, `screenshots/`, `ExtraResources/`) not created
- `fetched-epics.json` not initialized
- `projects.json` missing new flags

---

### ORC-002 — Register New Project: Minimal (All Optional Features Disabled)

**What is being tested:**
Registration flow with `has_epics: no`, `has_screenshots: no`, `has_extra_resources: no`.
Verifies optional folders and `fetched-epics.json` are NOT created.

**Pre-conditions:**
- `projects.json` does not contain the test project

**Steps:**
1. Load `@orchestrator` and type: `Start a new project`
2. Answer the registration questions:
   - Project name: `TEST-ORC-002`
   - Output directory: `{TEMP_PATH}/TEST-ORC-002`
   - Project key: `none`
   - Stories grouped under epics? → `no`
   - UI screenshots available? → `no`
   - Supplementary documentation? → `no`

**Expected behavior:**
- Orchestrator creates ONLY mandatory folders:
  `registry/`, `stories/raw/`, `stories/parsed/`, `context/`, `strategy/strategy-versions/`,
  `test-cases/`, `tracking/archive/`, `tracking/reviews/`, `config/`
- `epics/raw/`, `epics/parsed/`, `screenshots/`, `ExtraResources/` are NOT created
- `fetched-epics.json` is NOT initialized
- `projects.json` entry includes: `has_epics: false`, `has_screenshots: false`, `has_extra_resources: false`

**Failure modes:**
- Optional folders created when flags are false
- `fetched-epics.json` created despite `has_epics: false`

---

### ORC-003 — Register Project That Already Exists

**What is being tested:**
Step 2 of registration: "if project already exists: load it, skip to Run Startup."
Verifies the Orchestrator does not re-ask registration questions for a known project.

**Fixture:** `ORC-003-projects.fixture.json`

**Pre-conditions:**
1. Copy `ORC-003-projects.fixture.json` content → `projects.json` at workspace root
2. Ensure the output path in the fixture exists (create it if needed, even if empty)

**Steps:**
1. Load `@orchestrator` and type: `Start a new project`
2. When asked for project name, enter: `TEST-ORC-EXISTING`

**Expected behavior:**
- Orchestrator finds `TEST-ORC-EXISTING` in projects.json
- Skips all registration questions
- Proceeds directly to Run Startup: lists projects and asks which one to work with
- Does NOT overwrite or re-initialize any existing files

**Failure modes:**
- Re-asks registration questions for an existing project
- Overwrites `projects.json`, `pipeline-state.json`, or any other file

---

### ORC-004 — Run Menu: Invalid Input Rejected

**What is being tested:**
Menu input enforcement: "Accept ONLY a single digit 1–8. Any other input — including phrases — must be rejected."

**Pre-conditions:**
- At least one project registered in `projects.json`

**Steps:**
1. Load `@orchestrator`, select a project
2. At the "What would you like to do?" menu, type each of the following one at a time:
   - `run the pipeline`
   - `yes`
   - `0`
   - `9`
   - `full run`
   - ` ` (blank)

**Expected behavior:**
For each input:
- Orchestrator does NOT interpret it as any option
- Re-presents the full 1–8 menu
- Says: `"Please select an option by entering a number from 1 to 8."`

**Failure modes:**
- Any of the above inputs triggers a pipeline action
- Orchestrator infers intent and silently proceeds

---

### ORC-005 — Story Without Parent Epic When has_epics: true

**What is being tested:**
Fetcher behavior when `has_epics: true` but the fetched story has no parent field (or `parent: null`).
Verifies pipeline continues with NEEDS_REVIEW flag rather than halting.

**Pre-conditions:**
- Project registered with `has_epics: true`
- A story exists in the source system (or mock) with no parent/epic link

**Steps:**
1. Load `@orchestrator`, select the project, choose Option 1 (Full run)
2. Provide a story ID that has no parent epic

**Expected behavior:**
- Fetcher sets `epic_key: null` in the registry entry for that story
- Adds `flags: ["NEEDS_REVIEW"]` to the registry entry
- Reports inline: `"WARNING: Story {KEY} has no parent grouping entity. Saved with NEEDS_REVIEW flag."`
- Pipeline continues — story is NOT halted or skipped
- `epics/raw/` and `epics/parsed/` remain empty for this story (no epic file created)
- `fetched-epics.json` has no entry for a null epic key

**Failure modes:**
- Pipeline halts on a story with no epic
- Epic fetch is attempted with a null key
- NEEDS_REVIEW flag not set

---

### ORC-006 — Run Startup With No Projects Registered

**What is being tested:**
Orchestrator behavior when `projects.json` is empty or missing entirely.

**Pre-conditions:**
- `projects.json` at workspace root is either absent or contains `{ "projects": [] }`

**Steps:**
1. Load `@orchestrator`

**Expected behavior:**
- Orchestrator detects no registered projects
- Does NOT error or crash
- Offers: `"No projects registered yet. Would you like to register a new project? (yes / no)"`
- On `yes`: proceeds to project registration flow

**Failure modes:**
- Orchestrator crashes or throws an unhandled error
- Silently proceeds as if a project exists

---

### ORC-007 — Run Startup: Invalid Project Selection

**What is being tested:**
Project selection enforcement at Run Startup. The Orchestrator must reject inputs that are not valid project indices.

**Fixture:** `ORC-007-projects.fixture.json`

**Pre-conditions:**
1. Copy `ORC-007-projects.fixture.json` content → `projects.json` at workspace root (contains 3 projects)

**Steps:**
1. Load `@orchestrator`
2. At the project selection prompt, type each of the following one at a time:
   - `5` (out of range — only 3 projects exist)
   - `0`
   - `abc`
   - ` ` (blank)

**Expected behavior:**
- None of the above inputs selects a project
- Orchestrator re-presents the project list and asks again
- Does NOT proceed to the run menu

**Failure modes:**
- Any invalid input causes a project to be silently selected
- Orchestrator crashes on non-numeric input

---

### ORC-008 — Strategy Approved Before Context Re-Approval: Warning Shown

**What is being tested:**
Run Startup Step 5 (CONTEXT/STRATEGY VERSION CHECK): if `strategy_approved_at < context_approved_at`, the Orchestrator must warn that the strategy may be stale.

**Fixture:** `ORC-008-pipeline-state.fixture.json`

**Pre-conditions:**
1. Copy `ORC-008-pipeline-state.fixture.json` → `{PROJECT_OUTPUT}/registry/pipeline-state.json`
   (contains `context_approved_at` newer than `strategy_approved_at`)

**Steps:**
1. Load `@orchestrator` and select the project

**Expected behavior:**
- Orchestrator detects `strategy_approved_at < context_approved_at`
- Reports warning: `"Warning: the approved strategy (v{N}) was generated under a previous context version..."`
- Recommends re-running Story Prioritizer
- Asks: `"Continue anyway? (yes / no)"`
- On `yes`: proceeds to run menu
- On `no`: stops cleanly

**Failure modes:**
- Warning not shown
- Pipeline proceeds without asking

---

### ORC-009 — Duplicate Story IDs in Batch

**What is being tested:**
Batch deduplication: providing the same story key twice should not process it twice or create duplicate output.

**Pre-conditions:**
- At least one story registered with `status: "parsed"` in `fetched-stories.json`

**Steps:**
1. Load `@orchestrator`, select project, choose Option 1 (Full run)
2. When asked for story IDs, provide: `PROJ-101, PROJ-101, PROJ-102`

**Expected behavior:**
- Orchestrator deduplicates the list: treats it as `PROJ-101, PROJ-102`
- Processes each story exactly once
- Reports the deduplicated list before starting
- Does NOT generate two TC files for PROJ-101

**Failure modes:**
- Story processed twice (duplicate TC file or double state write)
- Orchestrator crashes or errors on duplicate input

---

### ORC-010 — Mixed Story Statuses in Same Batch

**What is being tested:**
Per-story routing: when stories in the same batch have different statuses, each is routed independently per the routing table.

**Pre-conditions:**
Manually set up `fetched-stories.json` with:
- `PROJ-101` → `status: "parsed"`
- `PROJ-102` → `status: "fetched"` (not yet parsed)
- `PROJ-103` → `status: "tc_generated"`

**Steps:**
1. Load `@orchestrator`, select project, choose Option 1 (Full run)
2. Provide story IDs: `PROJ-101, PROJ-102, PROJ-103`

**Expected behavior:**
- `PROJ-101` (`parsed`): runs Story Analysis → Context Build → Story Prioritizer → TC Generation
- `PROJ-102` (`fetched`): runs Parse → Story Analysis → Context Build → Story Prioritizer → TC Generation
- `PROJ-103` (`tc_generated`): skipped — reported as already complete
- Each story's route logged in the run log

**Failure modes:**
- All stories run the same steps regardless of status
- `tc_generated` story is reprocessed
- `fetched` story skips Parse

---

### ORC-011 — Batch ID Increments Correctly Within Same Day

**What is being tested:**
Batch ID generation: `batch-{YYYYMMDD}-NNN` counter increments per run on the same calendar day.

**Pre-conditions:**
- Project registered and pipeline-state.json initialized

**Steps:**
1. Start Run 1 — note the batch ID assigned (e.g. `batch-20260618-001`)
2. Complete or abort the run
3. Start Run 2 on the same day
4. Note the new batch ID
5. Start Run 3 on the same day

**Expected behavior:**
- Run 1: `batch-{today}-001`
- Run 2: `batch-{today}-002`
- Run 3: `batch-{today}-003`
- Counter stored in `pipeline-state.json → current_run.daily_batch_counter`
- Counter resets to `001` on the next calendar day

**Failure modes:**
- Batch ID repeated across runs
- Counter not persisted in pipeline-state.json
- Counter does not reset on new day

---

### ORC-012 — Output Path Missing at Run Startup

**What is being tested:**
Run Startup behavior when a registered project's `output_path` no longer exists on disk.

**Fixture:** `ORC-012-projects.fixture.json`

**Pre-conditions:**
1. Copy `ORC-012-projects.fixture.json` → `projects.json`
   (contains a project pointing to a path that does not exist)

**Steps:**
1. Load `@orchestrator`
2. Select the project with the missing path

**Expected behavior:**
- Orchestrator detects output path does not exist
- Reports: `"Output path for project '{name}' does not exist: {path}. Please verify the path or re-register the project."`
- Stops. Does NOT attempt to create folders or proceed with the run.

**Failure modes:**
- Pipeline proceeds with a missing output path
- Orchestrator auto-creates the folder without user confirmation

---

### ORC-013 — Option 8: Re-run Context Builder Flow

**What is being tested:**
Option 8 (Update project context): confirmation prompt, current version archived, new draft produced, Gate 2 re-approval required.

**Fixture:** `ORC-013-pipeline-state.fixture.json`

**Pre-conditions:**
1. Copy `ORC-013-pipeline-state.fixture.json` → `{PROJECT_OUTPUT}/registry/pipeline-state.json`
   (contains `context_approved: true`, `context_version: 1`, `strategy_approved: true`)
2. Ensure `{PROJECT_OUTPUT}/context/project-context.md` exists

**Steps:**
1. Load `@orchestrator`, select project, choose Option 8

**Expected behavior:**
- Orchestrator confirms: `"project-context.md is already approved. Re-running will archive the current version. Continue? (yes / no)"`
- On `yes`:
  - Archives current file as `project-context.v1.md`
  - Invokes Context Builder to produce new draft
  - Presents Gate 2 for re-approval
  - On approval: increments `context_version` to 2 in pipeline-state.json
- On `no`: stops cleanly, no files modified

**Failure modes:**
- Current `project-context.md` overwritten without archiving
- Gate 2 skipped after re-run
- `context_version` not incremented
