# TC Generator Agent Command

Read and execute the agent definition at `.claude/agents/tc-generator.md` exactly as written.

Apply the tool name mapping defined in CLAUDE.md (Tool Name Mapping table).

## Inputs

Story key to generate test cases for:
- If provided by the user (e.g., `/tc-generator TEST-FMT-001`): use that key
- If no key provided: ask the user which story to generate TCs for

To resolve `{PROJECT_OUTPUT}`:
1. Read `projects.json` at the workspace root
2. If only one project is registered: use it automatically
3. If multiple projects exist: ask the user to select one by name before proceeding

## Instructions

1. Follow the TC Generator's Role, Rules, Trigger Conditions, and Execution Steps from `.claude/agents/tc-generator.md` exactly
2. Apply all 11 Global Rules from CLAUDE.md
3. Apply all Path Schema rules from CLAUDE.md
4. Check prerequisites: `context_approved: true` and `strategy_approved: true` in `pipeline-state.json` (HALT if not approved)
5. Read parsed story from `{PROJECT_OUTPUT}/stories/parsed/{STORY-KEY}.parsed.json`
6. Load screenshots from:
   - If `has_epics: true`: `screenshots/{EPIC-KEY}/`
   - If `has_epics: false`: `screenshots/{STORY-KEY}/`
7. Load ExtraResources from:
   - Root level: `ExtraResources/{filename}` (applies to all stories)
   - Epic level (if has_epics: true): `ExtraResources/{EPIC-KEY}/{filename}` (scoped to that epic)
   - Story level (if has_epics: false): `ExtraResources/{STORY-KEY}/{filename}` (scoped to that story)
8. Generate one test case per AC:
   - Standard TC: numbered steps with preconditions, actions, expected result, pass/fail
   - Observation TC: generated for open questions or indeterminate expected results
   - Blocked TC: generated when AC is untestable (contradicts out_of_scope or other AC)
9. Save TCs to `{PROJECT_OUTPUT}/test-cases/{STORY-KEY}-test-cases.csv`
10. Present draft TCs for user approval via Gate 4
11. Log all findings via `assumption-tracker` (missing UI elements, design mismatches, etc.)

## Critical Rules for TC Generator

- **Never invent UI element locations** — if a button/field is referenced but not shown in any screenshot or design file, log as A-NNN assumption and reference it in TC steps
- **Never self-resolve discrepancies** — if there's an open D-NNN (AC vs. screenshot), generate Observation TC citing the discrepancy
- **Respect blocked ACs** — if an AC is marked `testable: false` (blocked by a question or out_of_scope), generate a BLOCKED TC explaining why
- **Cache screenshots** — if multiple stories in the same batch reference the same epic, load the screenshot folder once and cache it. Do NOT re-scan for the same SCOPE-KEY
- **Handle has_epics correctly**:
  - If `has_epics: true` and `epic_key: null`: skip epic subfolder step, use story-level ExtraResources only
  - If `has_epics: false` and `epic_key: null`: skip epic subfolder step entirely, use story-level ExtraResources only, use story screenshots only

## Exception Handling

Follow the Exception Handling table in `.claude/agents/tc-generator.md` for:
- Context not approved (halt at Step 1)
- Strategy not approved (halt at Step 1)
- Parsed story not found (suggest running Parser first)
- Screenshots not found (generate TCs from AC text only)
- ExtraResources not found (generate TCs without resource constraints)
- has_epics: false project (never access epics/ or epic-level ExtraResources)
