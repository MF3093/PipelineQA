# Category 2 — Malformed Story Input Tests

Tests how the Parser and Fetcher handle stories that violate expected structure.

## How to Run

1. Copy the desired `MAL-NNN.raw.json` to `{PROJECT_OUTPUT}/stories/raw/`.
2. Add a matching entry in `fetched-stories.json` with `status: "fetched"`.
3. Load `@parser` and instruct it to parse the story key.
4. Observe flags, pauses, and output.

## Test Index

| File | Test ID | Condition | Agent | Expected Behavior |
|------|---------|-----------|-------|-------------------|
| MAL-001.raw.json | MAL-001 | `description` is `null` | Parser | `description` set to `summary`; `acs: []`; `needs_review: true`; `flags: ["NEEDS_REVIEW"]` |
| MAL-002.raw.json | MAL-002 | Story body has no AC section (no list, no AC header) | Parser | Pause and ask user to identify AC section; `NEEDS_REVIEW` if user cannot confirm |
| MAL-003.raw.json | MAL-003 | `parent` field is `null` (no epic) | Fetcher | `NEEDS_REVIEW` flag set on registry entry; continues without epic fetch; warning shown |
| MAL-004.raw.json | MAL-004 | Contradictory ACs — AC-4 says element "visible AND hidden" | Parser | `AC_QUALITY_ISSUE` flag; both conflicting ACs flagged; `needs_review: true`; pause before write |
| MAL-005.raw.json | MAL-005 | Story title contains "Spike" | Parser | `flags: ["SPIKE"]` set |
| MAL-006.raw.json | MAL-006 | Multiple ACs contain vague/unmeasurable terms | Parser | `AC_QUALITY_ISSUE` flag on vague ACs; Q-NNN logged via assumption-tracker; 3-choice pause shown |
| MAL-007.raw.json | MAL-007 | Extremely long description (single paragraph >2000 chars) | Parser | PowerShell fallback used; story parsed successfully; `NEEDS_REVIEW` only if PS fails |

## MAL-001 vs MAL-002 Distinction

- **MAL-001**: `description` field is literally `null` in the JSON payload — the field is absent/null.
- **MAL-002**: `description` field has text content but contains **no AC section** — no numbered list, no "Acceptance Criteria" header, no Given/When/Then.

## MAL-004 Detail

AC-4 text: `"The Import button is visible AND hidden on the upload page depending on file type selected."`
This is a self-contradictory AC (visible AND hidden). The Parser must detect this as conflicting language
and set `AC_QUALITY_ISSUE`, not attempt to extract it as a conditional.

## MAL-006 Vague ACs to Flag

The following ACs in MAL-006 contain vague/unmeasurable terms and must each trigger AC_QUALITY_ISSUE:
- AC-5: `"The system works correctly when the user submits the form."`
- AC-6: `"The form is properly validated before submission."`
- AC-7: `"The save button functions as expected in all scenarios."`

ACs 1–4 are well-formed and should NOT be flagged.

## Pass / Fail Criteria

**Pass:** System behaves exactly as described in Expected Behavior column above.
**Fail:** System processes malformed content as if it were valid, silently ignores flags, or crashes.
**Partial:** Correct flag set but user-facing message is unclear or missing.
