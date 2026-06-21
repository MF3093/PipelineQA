# Story Analyzer Agent Command

Read and execute the agent definition at `.github/agents/story-analyzer.agent.md` exactly as written.

Apply the tool name mapping from CLAUDE.md:
- `read_file` → `Read` tool
- `grep_search` → `Grep` tool (NEVER on `fetched-stories.json`, `fetched-epics.json`, or `pipeline-state.json` — use `Read` tool only)
- `file_search` → `Glob` tool
- `run_in_terminal` → `Bash` tool
- `edit_file` → `Edit` tool
- `create_file` → `Write` tool

## Inputs

Story key(s) to analyze:
- If provided by the user (e.g., `/story-analyzer TEST-FMT-001`): use that key
- If no key provided: ask the user which story to analyze

To resolve `{PROJECT_OUTPUT}`:
1. Read `projects.json` at the workspace root
2. If only one project is registered: use it automatically
3. If multiple projects exist: ask the user to select one by name before proceeding

## Instructions

1. Follow the Story Analyzer's Role, Rules, Trigger Conditions, and Execution Steps from `.github/agents/story-analyzer.agent.md` exactly
2. Apply all 10 Global Rules from CLAUDE.md
3. Apply all Path Schema rules from CLAUDE.md
4. Read the parsed story JSON from `{PROJECT_OUTPUT}/stories/parsed/{STORY-KEY}.parsed.json`
5. Analyze for:
   - Discrepancies (AC vs. description, AC vs. AC, AC vs. screenshots, AC vs. Out of Scope)
   - Coverage gaps (missing error behaviors, validation rules without error paths)
   - Vague or unmeasurable ACs
   - Questions blocking test generation
6. Log all findings via `assumption-tracker` with appropriate types (Discrepancy, Question, Blocker, etc.)
7. Report analysis results to the user
8. Do NOT modify the parsed story JSON — only read and analyze

## Key Analysis Tasks

- **Contradiction detection** — flag when two ACs describe opposite outcomes for the same scenario
- **Vague language detection** — flag when ACs use unmeasurable terms without specific criteria
- **Screenshot comparison** — if screenshots exist, compare AC descriptions against visual content
- **Out of Scope validation** — flag if ACs describe behavior marked as out of scope
- **Error behavior validation** — flag if validation rules are specified without error messages
- **Design reference tracking** — note if ACs reference design elements not shown in any design file or screenshot

## Exception Handling

Follow the Exception Handling table in `.github/agents/story-analyzer.agent.md` for:
- Parsed file not found (suggest running Parser first)
- Epic parent story missing (analyze without epic context)
- Screenshots not found (analyze story-only content)
- ExtraResources not found (analyze without resource constraints)
