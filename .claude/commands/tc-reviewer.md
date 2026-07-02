# TC Reviewer Agent Command

Read and execute the agent definition at `.github/agents/tc-reviewer.agent.md` exactly as written.

Apply the tool name mapping from CLAUDE.md:
- `read_file` → `Read` tool
- `grep_search` → `Grep` tool (NEVER on `fetched-stories.json`, `fetched-epics.json`, or `pipeline-state.json` — use `Read` tool only)
- `file_search` → `Glob` tool
- `run_in_terminal` → `Bash` tool
- `edit_file` → `Edit` tool
- `create_file` → `Write` tool

## Inputs

Review mode:
- If the user provides a mode (e.g., `/tc-reviewer cross-story` or `/tc-reviewer integration`): use that mode
- If no mode provided: ask the user to select:
  - **Cross-story** — detect redundancy and subset coverage across stories
  - **Integration** — detect contradictory expected results for the same component/action
  - **Both** — run both analyses

To resolve `{PROJECT_OUTPUT}`:
1. Read `projects.json` at the workspace root
2. If only one project is registered: use it automatically
3. If multiple projects exist: ask the user to select one by name before proceeding

## Instructions

1. Follow the TC Reviewer's Role, Rules, Trigger Conditions, and Execution Steps from `.github/agents/tc-reviewer.agent.md` exactly
2. Apply all 11 Global Rules from CLAUDE.md
3. Apply all Path Schema rules from CLAUDE.md
4. Check prerequisite: at least 2 story TC files must exist in `{PROJECT_OUTPUT}/test-cases/` (HALT if fewer than 2)
5. Read all TC files from `{PROJECT_OUTPUT}/test-cases/{STORY-KEY}-test-cases.csv`
6. Read priority matrix from `{PROJECT_OUTPUT}/strategy/priority-matrix.md` (for story context)
7. Run analysis per the user's selected mode:
   - **Cross-story**: detect subset relationships (all assertions of TC-A are observable in TC-B)
   - **Integration**: detect contradictory expected results for the same component/action
8. Report findings:
   - Type 1 (Redundancy): "TC-A is redundant with TC-B — same assertions"
   - Type 2 (Subset): "TC-A is a subset of TC-B — assertions of A are observable in B"
   - Type 3 (Contradiction): "TC-A expects X; TC-B expects opposite Y for the same component/action"
9. Present recommendations:
   - Redundancy: "Consider consolidating these TCs"
   - Subset: "Consider absorbing TC-A into TC-B or removing TC-A"
   - Contradiction: "These TCs describe incompatible outcomes. Confirm intentional evolution or resolve conflict."
10. Save report to `{PROJECT_OUTPUT}/tracking/reviews/cross-story-review-{batch_id}.md` or `integration-review-{batch_id}.md`
11. **CRITICAL**: Do NOT modify any TC file. Reports and recommendations only.

## Review Modes

### Cross-Story Mode

- Compare all TCs across all stories in the project
- Detect: Exact duplicates, subsets, functional overlap
- For each finding:
  - Report the two TC IDs and story keys
  - Quote the assertion/step from both TCs
  - Recommend consolidation or removal

### Integration Mode

- Compare expected results across stories for the same component/action
- Detect: Mutually exclusive outcomes for the same user interaction
- For each finding:
  - Report the two TC IDs and story keys
  - Quote the exact conflicting expected results
  - Assess: intentional evolution (different feature phases), genuine contradiction, or misunderstanding

## Exception Handling

Follow the Exception Handling table in `.github/agents/tc-reviewer.agent.md` for:
- Fewer than 2 TC files (halt with message about minimum requirement)
- TC file format errors (skip malformed files, report which ones)
- Missing priority matrix (proceed with review, note that context is unavailable)
- Report file already exists (ask user whether to overwrite or append)
