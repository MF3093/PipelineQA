# Category 3 — State Corruption & Crash Recovery Tests

Tests pipeline resilience when registry files are corrupted or sessions crash mid-run.

> These tests require **direct filesystem manipulation** — no mock JSON fixtures are used.
> Follow each step exactly. Do not skip the verification step.

---

## CRA-001 — Kill Session Mid TC Generation

**Goal:** Verify crash recovery detects an in-progress story on next startup.

**Steps:**
1. Start a full pipeline run with at least 2 stories.
2. During TC generation for the first story (after the batch plan is approved, before Gate 4), close the Copilot Chat panel or end the VS Code session.
3. Manually open `{PROJECT_OUTPUT}/registry/pipeline-state.json` and verify `current_run.current_story` is set to the story key that was in progress.
4. Open a new Copilot Chat session and load `@orchestrator`.
5. Select the same project.

**Expected:**
- Orchestrator detects `current_story` is non-null.
- Reports: "Story {KEY} was in progress when the previous session ended. TC output may be incomplete."
- Asks: "Reprocess {KEY} from the start? (yes / skip)"
- On "yes": resets that story's status to "parsed" and clears `current_story`.

---

## CRA-002 — Delete pipeline-state.json Mid-Run

**Goal:** Verify the prereq checker stops cleanly when the state file is missing.

**Steps:**
1. Locate `{PROJECT_OUTPUT}/registry/pipeline-state.json`.
2. Rename or delete the file.
3. Load `@orchestrator` and attempt to start a run.

**Expected:**
- Orchestrator reports missing prereq: "Pipeline state file not found."
- Stops. Does not proceed to any agent.
- Does NOT attempt to auto-recreate the file.

---

## CRA-003 — Stale Lock (lock.locked = true)

**Goal:** Verify the Orchestrator detects and offers to release a stale lock.

**Setup:**
1. Open `{PROJECT_OUTPUT}/registry/pipeline-state.json`.
2. Set `"locked": true` and `"locked_at": "2026-06-07T08:00:00Z"` (yesterday's timestamp).
3. Save the file.

**Steps:**
1. Load `@orchestrator` and start a new run.

**Expected:**
- Orchestrator detects the lock and shows:
  "A session is currently active (locked since 2026-06-07T08:00:00Z). If this is a stale lock from a crashed session, you can force-unlock. Force unlock and continue? (yes / no)"
- On "yes": sets `locked: false`, proceeds normally.
- On "no": STOP, does not proceed.

---

## CRA-004 — Registry Says "parsed" But Parsed File Missing

**Goal:** Verify the Orchestrator's reconciliation check catches status/file mismatches.

**Setup:**
1. Confirm a story with `status: "parsed"` exists in `fetched-stories.json`.
2. Delete its corresponding `{PROJECT_OUTPUT}/stories/parsed/{STORY-KEY}.parsed.json` file.

**Steps:**
1. Load `@orchestrator` and start a run (any option).

**Expected:**
- Orchestrator reports: "Registry inconsistency detected: {KEY}: status is 'parsed' but expected file is missing."
- Asks: "Reset status to 'fetched' to allow reprocessing? (yes / no)"
- On "yes": resets status in registry, allows reprocessing.

---

## CRA-005 — Corrupt fetched-stories.json (Invalid JSON)

**Goal:** Verify the Orchestrator stops and does not attempt auto-repair on a corrupt registry.

**Setup:**
1. Open `{PROJECT_OUTPUT}/registry/fetched-stories.json`.
2. Insert invalid JSON (e.g., delete the closing `}` or add `CORRUPTED` text at the top).
3. Save the file.

**Steps:**
1. Load `@orchestrator` and attempt to start a run.

**Expected:**
- Orchestrator reports: "Corrupt registry file: fetched-stories.json cannot be parsed."
- STOP. Does not proceed. Does NOT attempt to auto-repair or overwrite.
- User must manually restore the file.

---

## Cleanup After Each Test

After each CRA test, restore the file to its original state before running the next test.
Keep a backup copy of your `registry/` folder before starting this category.
