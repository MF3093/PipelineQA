# Parsed Story Fixtures

Pre-built `ParsedStory` JSON files for testing TC Generator, TC Reviewer, and related agent behaviors
without running a full Phase 1 pipeline.

## How to Use

1. Copy the desired `.parsed.json` to `{PROJECT_OUTPUT}/stories/parsed/`.
2. Ensure a matching entry exists in `fetched-stories.json` with `status: "parsed"`.
3. Ensure `project-context.md` and `priority-matrix.md` are approved in your test project output.
4. Load `@tc-generator` and instruct it to generate TCs for the story key.

## Fixture Index

| File | Story Key | Purpose | Key Test Scenarios |
|------|-----------|---------|-------------------|
| `TEST-TCG-001.parsed.json` | TEST-TCG-001 | Standard well-formed story with 4 ACs | TCG-H1, GAP-D1, GAP-D2 |
| `TEST-TCG-SPIKE.parsed.json` | TEST-TCG-SPIKE | Story flagged SPIKE | TCG-N5 |
| `TEST-TCG-OBSERVATION.parsed.json` | TEST-TCG-OBSERVATION | Story with open Q-NNN → Observation TC | TCG-N7, TCG-N8 |
| `TEST-TCG-CONDITIONAL.parsed.json` | TEST-TCG-CONDITIONAL | Story with conditional ACs + timing AC | TCG-N11, TCG-N12, TCG-N13, TCG-N14 |

## Key Assertions by Fixture

### TEST-TCG-001 (standard)
- 4 TCs minimum expected (one per AC)
- AC-3 is conditional → 2 TCs: empty state + non-empty state
- Output CSV columns must match `Import Format` from `project-context.md` exactly (GAP-D1)
- Multi-step TCs must have blank Title/Priority/Description on continuation rows (GAP-D2)

### TEST-TCG-SPIKE
- TC Generator must warn and ask yes/no before generating
- If user answers "no": no TC file created
- If user answers "yes": TCs generated with spike disclaimer logged

### TEST-TCG-OBSERVATION
- AC-1 and AC-2: standard TCs generated
- AC-3 (`testable: false`, `blocked_by: "Q-001"`): **Observation TC** generated
  - `Test Type = Observation`
  - `Priority = Low`
  - `Expected Result` starts with `"OBSERVATION — no pass/fail."`
  - Precondition includes: `"NOTE: This TC has no pass/fail verdict."`

### TEST-TCG-CONDITIONAL
- AC-1 (conditional): 2 TCs — State A (Grounded → modal shown) + State B (not Grounded → no modal)
- AC-3 (conditional with side effect — modal action buttons trigger navigation): +2 transition TCs
- AC-4 (timing/performance): **own TC, never absorbed as Pattern B**
- Total TCs: ≥ 6
