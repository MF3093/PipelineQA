# Category 4 — Gate Bypass Attempts

Tests whether the system can be tricked into skipping approval gates or prerequisite checks.

> These tests require a **live pipeline run** against real (or mock) story data.
> Run them sequentially during an active session.

---

## BYP-001 — "Yes to All Gates" at Run Startup

**Goal:** Verify that no batch-approve mechanism exists — each gate must be confirmed individually.

**Steps:**
1. Start a full pipeline run with `@orchestrator`.
2. At the very beginning (run startup), type: `"yes to all gates"` or `"approve everything"`.

**Expected:**
- The Orchestrator does NOT interpret this as a blanket approval.
- It proceeds to the first applicable gate (Gate 2 — context, or Gate 3 — strategy) and presents it individually.
- The input is either ignored or prompted to re-enter with: `"Please select an option by entering a number from 1 to 8."`

---

## BYP-002 — Unrecognized Input at a Gate

**Goal:** Verify that unrecognized inputs at an approval gate do NOT advance the pipeline.

**Steps:**
1. During an active pipeline run, reach Gate 2 (after context build) or Gate 3 (after strategy generation).
2. At the gate prompt, type: `"skip"` (not in the approved response list).

**Expected:**
- Gate stays open.
- Orchestrator responds: `"Response not recognized. Please reply with: yes / edit / reject"`
- Pipeline does NOT advance.
- Test other unrecognized inputs: `"continue"`, `"proceed"`, `"done"`, `"sure"`, `"ok then"`.

---

## BYP-003 — Request TC Generation for a Story with Status "fetched"

**Goal:** Verify the Orchestrator blocks TC generation when Phase 1 has not been completed.

**Setup:**
1. Fetch one or more stories (status = "fetched") but do NOT run Parse.

**Steps:**
1. Load `@orchestrator` and select Option 3 (Phase 2 only) with those story IDs.

**Expected:**
- Orchestrator reports: `"Story {KEY} has not been fetched yet. Run Phase 1 first."`
  (or equivalent — the story has status "fetched", not "parsed")
- TC generation does NOT begin.
- No TC files are created.

---

## BYP-004 — Request Strategy Without Approved Context

**Goal:** Verify the Orchestrator blocks Strategy when context_approved = false.

**Setup:**
1. Open `{PROJECT_OUTPUT}/registry/pipeline-state.json`.
2. Set `"context_approved": false`.
3. Save the file.

**Steps:**
1. Load `@orchestrator` and select Option 6 (Strategy only).

**Expected:**
- Orchestrator reports: `"project-context.md must be approved before the strategy can be generated."`
- STOP. Strategy agent is NOT invoked.

---

## Pass / Fail Criteria

**Pass:** Pipeline does NOT advance on any of the above inputs.
**Fail:** Pipeline skips a gate, advances without explicit approval, or generates any output.
**Partial:** Gate blocks correctly but error message is absent or unclear.
