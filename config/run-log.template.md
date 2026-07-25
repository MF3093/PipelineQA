# Run Log — {RUN_ID}

> Auto-generated at run startup. Updated at each phase transition and gate decision.

---

## Run Metadata

- **Run ID:** {RUN_ID}
- **Batch ID:** {BATCH_ID}
- **Project:** {PROJECT_NAME}
- **Started at:** {TIMESTAMP}
- **Run type:** {Full | Phase 1 | Phase 2 | Fetch only | Parse only | Strategy only | TC Gen only}
- **Story keys:** {comma-separated list}
- **Completed at:** {TIMESTAMP or IN PROGRESS}
- **Final status:** {completed | paused | aborted}

---

## Phase Log

| # | Phase | Started | Finished | Result | Notes |
|---|-------|---------|----------|--------|-------|
| 1 | Fetch | {time} | {time} | {completed / skipped} | {new stories: N, new epics: N, skipped: N} |
| 2 | Parse | {time} | {time} | {completed / partial} | {flags raised} |
| 3 | Story Analysis | {time} | {time} | {completed} | {Q/D entries logged} |
| 4 | Context Build | {time} | {time} | {approved / skipped} | {first run or reuse} |
| 5 | Story Prioritizer | {time} | {time} | {approved / rejected / skipped} | {new or extension} |
| 6 | TC Generation | {time} | {time} | {approved / partial / rejected} | {TC count} |

### Fetch Detail

```
NEW STORIES:
| # | Story Key | Summary | Epic | Has Description | Comments | Flags |
|---|-----------|---------|------|-----------------|----------|-------|
| {N} | {KEY} | {summary} | {EPIC-KEY or —} | {Yes / No} | {N} | {— / NEEDS_REVIEW} |

NEW EPICS:
| # | Epic Key | Summary | Stories Fetched |
|---|----------|---------|-----------------|
| {N} | {EPIC-KEY} | {summary} | {N} |

SKIPPED (already fetched): {N} stories, {N} epics
```
### Parse Detail

```
Story Parser - Batch {BATCH_ID}
--------------------------------
Stories parsed:          {N}
Total ACs extracted:     {N}
  Testable:              {N}
  Blocked (untestable):  {N}
Flagged stories:         {keys and flag types, or "none"}
AC quality issues:       {stories with AC_QUALITY_ISSUE flag, or "none"}
  Conflicting language:  {AC-N in STORY-KEY, or "none"}
  Vague terms:           {AC-N in STORY-KEY, or "none"}
  Design mismatches:     {AC-N in STORY-KEY, or "none"}
Stories needing review:  {keys, or "none"}
Open items logged:       {N} -> tracking/assumptions.md
--------------------------------
```
---

## Gate Decisions

### Gate 2 — Context Approval
- **Decision:** {yes | edit | reject | skipped (already approved)}
- **Edits applied:** {none | fields corrected}
- **Corrections logged:** {COR-NNN references or none}

### Gate 3 — Strategy Approval
- **Decision:** {yes | edit | reject | skipped}
- **Edits applied:** {none | sections corrected}
- **Corrections logged:** {COR-NNN references or none}

### Gate 4 — TC Approval
| Story Key | Decision | Edits Applied | Corrections Logged |
|-----------|----------|---------------|-------------------|
| {KEY} | {yes / edit / reject / skip} | {description or none} | {COR-NNN or none} |

---

## Quality Signals

- **Parsing quality:** {Standard: N, Relaxed: N, Heuristic: N, Failed: N}
- **Open assumptions at run end:** {A: N, Q: N, B: N, D: N}
- **Resolved this run:** {N}
- **Archived this run:** {N}
- **TC count validation:** {all match | mismatches: list}

---

## Incidents

<!-- Record any unexpected behaviors, errors, or agent failures during this run -->

