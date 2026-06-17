# Skill: assumption-tracker

## Invocation — Log Entry

```
assumption-tracker.invoke({
  type: "Assumption" | "Question" | "Blocker" | "Discrepancy",
  story_key: string,
  ac_id: string | null,          // e.g. "AC-1" — null if not AC-specific
  description: string,           // what is assumed, unknown, blocked, or mismatched
  impact: string,                // what breaks or is uncertain if this is wrong
  batch_id: string,              // current batch ID from pipeline-state.json
  // Required only when type = "Discrepancy":
  screenshot_filename?: string,
  what_prototype_shows?: string,
  what_ac_says?: string
}) → { id: string, is_duplicate: boolean }
```

---

## Invocation — Resolve Entry (D3-1)

Callable by the user at any time, or by any agent that discovers a resolution for an existing open entry during its run.

```
assumption-tracker.resolve({
  id: string,                    // e.g. "Q-001"
  resolution: string             // brief explanation of how it was resolved
}) → { success: boolean, message: string }
```

**Resolve behavior:**
1. Find the entry with matching ID in the Open Items section. If not found: return `{ success: false, message: "ID {id} not found in Open Items." }`
2. Cut the entry block from Open Items. Update: `**Status:** Resolved`, `**Resolved Date:** today`, `**Resolution:** {text}`. Paste into Resolved Items.
3. Re-read Open Items to confirm entry is gone. If still present: repeat.
4. Run Post-Write Verification.
5. Return `{ success: true, message: "Entry {id} resolved and moved to Resolved Items." }`

> **Forbidden:** Updating only the Status field in place is not a valid resolution. The entry must be physically absent from Open Items. No direct file edits to Status fields — always call `assumption-tracker.resolve()`.

---

## Invocation — Log Already-Resolved Entry

Use this path when an agent knows at write time that an item is already resolved (e.g., a discrepancy confirmed and resolved within the same run via comments or a client decision applied inline).

```
assumption-tracker.invoke_resolved({
  type: "Assumption" | "Question" | "Blocker" | "Discrepancy",
  story_key: string,
  ac_id: string | null,
  description: string,
  impact: string,
  batch_id: string,
  resolution: string,
  resolved_date: string,           // YYYY-MM-DD
  // Required only when type = "Discrepancy":
  screenshot_filename?: string,
  what_prototype_shows?: string,
  what_ac_says?: string
}) → { id: string }
```

**Behavior:**
1. Run duplicate detection. If duplicate found: return existing ID — do not write.
2. Generate next ID (same rules as `invoke()`).
3. Write entry **directly into `## Resolved Items`** with Status: Resolved, Resolved Date, and Resolution fields.
4. Run Post-Write Verification.
5. Return `{ id: new_id }`.

> **Forbidden:** Do not use `invoke()` followed by `resolve()` as a substitute for this path.

---

## ID Generation

1. Read `{PROJECT_OUTPUT}/tracking/assumptions.md` (active file only).
2. Find the highest numeric suffix per type prefix across both sections:
   - `A-` Assumption | `Q-` Question | `B-` Blocker | `D-` Discrepancy
3. Next ID = prefix + zero-padded 3-digit number (e.g. `A-001`, `Q-002`).
4. IDs are globally unique within the project — not per story or batch.
5. If `assumptions.md` does not exist: create it with the standard header, start IDs at `001`.

---

## Duplicate Detection

Before writing, scan all entries (Open and Resolved) for a match where ALL of the following are equal (case-insensitive): `type`, `story_key`, `ac_id`, and first 80 chars of `description`.
- Match found: return `{ id: existing_id, is_duplicate: true }`. Do not write.
- Same `story_key + ac_id + type` but different description: create new entry.
- If uncertain: always create.

Each entry header includes a duplicate-detection comment:
```markdown
### {ID} — {type} <!-- match: {type}|{story_key}|{ac_id}|{description[:80]} -->
```

---

## Entry Written to assumptions.md

### Standard entry (all types) — invoke():

```markdown
### {ID} — {type} <!-- match: {type}|{story_key}|{ac_id}|{description[:80]} -->
- **Story:** {story_key}
- **AC:** {ac_id or "N/A"}
- **Batch:** {batch_id}
- **Description:** {description}
- **Impact:** {impact}
- **Status:** Open
- **Created:** {YYYY-MM-DD}
- **Resolved Date:** —
- **Resolution:** —
```

**Placement:** Always appended under `## Open Items`. Never insert a new `invoke()` entry under `## Resolved Items`.

### Additional fields for type = Discrepancy:

```markdown
- **Screenshot:** {screenshot_filename}
- **Prototype Shows:** {what_prototype_shows}
- **AC Says:** {what_ac_says}
```

---

## Post-Write Verification (mandatory after every write)

1. Re-read full `assumptions.md`.
2. Every entry under `## Open Items` must have `**Status:** Open`. If any has `Resolved`: move to Resolved Items.
3. Every entry under `## Resolved Items` must have `**Status:** Resolved`. If any has `Open`: move to Open Items.
4. Repeat until no mismatches remain, then return to calling agent.

---

## File Structure of assumptions.md

```markdown
# Tracking: Assumptions, Questions, Blockers, Discrepancies

## Legend
| Prefix | Type        | Meaning                                              |
|--------|-------------|------------------------------------------------------|
| A-NNN  | Assumption  | Something assumed true without explicit confirmation |
| Q-NNN  | Question    | Information needed that is currently unknown         |
| B-NNN  | Blocker     | TC cannot be written due to a conflict or constraint |
| D-NNN  | Discrepancy | Prototype shows something not stated in the story AC |

---

## Open Items

<!-- Entries added here by assumption-tracker, newest first within each run -->

---

## Resolved Items

<!-- Entries moved here by assumption-tracker.resolve() or manually by QA Lead -->
```

---

## Batch Archiving (A2 — controls file growth)

Invoked by the Orchestrator at the end of every completed run (status = "completed").

```
assumption-tracker.archive({
  batch_id: string,              // batch to archive
  project_output: string
}) → { archived_count: number, archive_path: string }
```

**Archive behavior:**
1. Read `tracking/assumptions.md`. Collect ALL entries in the `## Resolved Items` section, regardless of batch.
2. If none: return `{ archived_count: 0, archive_path: null }`. Do nothing.
3. Write all collected entries to `tracking/archive/assumptions-{batch_id}.md` (grouped by batch within the file if multiple batches are present).
4. Remove all archived entries from the Resolved Items section of `assumptions.md`.
5. Add reference line: `<!-- {N} resolved entries archived to archive/assumptions-{batch_id}.md on {YYYY-MM-DD} -->`
6. Return `{ archived_count: N, archive_path: "tracking/archive/assumptions-{batch_id}.md" }`.

Not archived: open items (regardless of batch). All resolved items are archived on every run.

---

## Permissions

- Read: `{PROJECT_OUTPUT}/tracking/assumptions.md`
- Write: `{PROJECT_OUTPUT}/tracking/assumptions.md`
- Write: `{PROJECT_OUTPUT}/tracking/archive/assumptions-{batch_id}.md` (archive only)
- Cannot read or write any other file.

---

## Rules That Apply

- Global Rule 2 (Never Invent Information) — this skill is the mechanism for Rule 2 compliance.
- Global Rule 8 (Assumptions Are Mandatory Outputs) — calling agents must invoke this skill
  before presenting any deliverable that contains uncertainty.
