# Skill: assumption-tracker

## Invocation — Log Entry

```
assumption-tracker.invoke({
  type: "Assumption" | "Question" | "Blocker" | "Discrepancy",
  story_key: string,
  ac_id: string | null,
  description: string,
  impact: string,
  batch_id: string,
  screenshot_filename?: string,       // required only for Discrepancy
  what_prototype_shows?: string,      // required only for Discrepancy
  what_ac_says?: string               // required only for Discrepancy
}) → { id: string, is_duplicate: boolean }
```

Entry is appended under `## Open Items`. Never under Resolved Items.

## Invocation — Resolve Entry

Callable by any agent or user. Moves entry from Open → Resolved.

```
assumption-tracker.resolve({
  id: string,
  resolution: string
}) → { success: boolean, message: string }
```

1. Find entry by ID in Open Items. Not found → return failure.
2. Cut from Open Items → set `Status: Resolved`, `Resolved Date: today`, `Resolution: {text}` → paste into Resolved Items.
3. Re-read to confirm entry absent from Open Items. Run Post-Write Verification.

> **Forbidden:** Updating Status in place is not valid. Entry must be physically moved. Always use `resolve()`.

## Invocation — Log Already-Resolved Entry

For items known-resolved at write time (e.g., discrepancy resolved within same run).

```
assumption-tracker.invoke_resolved({
  ...same params as invoke(),
  resolution: string,
  resolved_date: string               // YYYY-MM-DD
}) → { id: string }
```

Writes directly into `## Resolved Items` with Status: Resolved. Runs duplicate detection first. Do not use `invoke()` + `resolve()` as substitute.

## ID Generation

| Prefix | Type |
|--------|------|
| `A-` | Assumption |
| `Q-` | Question |
| `B-` | Blocker |
| `D-` | Discrepancy |

Read `{PROJECT_OUTPUT}/tracking/assumptions.md` → find highest numeric suffix per prefix across both sections → next ID = prefix + zero-padded 3 digits (e.g., `A-001`). IDs are globally unique within the project. If file missing: create with standard header, start at `001`.

## Duplicate Detection

Before writing, scan all entries (Open + Resolved). Match = all equal (case-insensitive): `type`, `story_key`, `ac_id`, first 80 chars of `description`.
- Match found → return existing ID, do not write.
- Same `story_key + ac_id + type` but different description → create new entry.

Each entry header includes: `### {ID} — {type} <!-- match: {type}|{story_key}|{ac_id}|{description[:80]} -->`

## Entry Format

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

Discrepancy adds: `**Screenshot:** {filename}`, `**Prototype Shows:** {text}`, `**AC Says:** {text}`

## Post-Write Verification

After every write: re-read `assumptions.md`. Any entry under Open Items with `Status: Resolved` → move to Resolved Items (and vice versa). Repeat until no mismatches.

## File Structure (assumptions.md)

```markdown
# Tracking: Assumptions, Questions, Blockers, Discrepancies

## Open Items
<!-- Entries added here by assumption-tracker, newest first within each run -->

## Resolved Items
<!-- Entries moved here by resolve() or manually by QA Lead -->
```

## Batch Archiving

Invoked by Orchestrator at end of completed run.

```
assumption-tracker.archive({ batch_id: string, project_output: string })
→ { archived_count: number, archive_path: string }
```

1. Collect ALL entries from `## Resolved Items` (regardless of batch). If none → return 0, do nothing.
2. Write to `tracking/archive/assumptions-{batch_id}.md` → remove from Resolved Items → add reference comment.

Open items are never archived.

## Permissions & Rules

| Permission | Path |
|------------|------|
| Read/Write | `{PROJECT_OUTPUT}/tracking/assumptions.md` |
| Write | `{PROJECT_OUTPUT}/tracking/archive/assumptions-{batch_id}.md` |

Cannot read or write any other file. Implements Global Rule 2 (Never Invent) and Rule 8 (Assumptions Are Mandatory Outputs).
