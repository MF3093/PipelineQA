# Story Prioritizer Agent Command

Read and execute the agent definition at `.claude/agents/story-prioritizer.md` exactly as written.

Apply the tool name mapping defined in CLAUDE.md (Tool Name Mapping table).

## Inputs

Story key(s) to prioritize:
- If provided by the user (e.g., `/story-prioritizer TEST-FMT-001 TEST-FMT-002`): use those keys
- If no keys provided: ask the user which story/stories to prioritize

To resolve `{PROJECT_OUTPUT}`:
1. Read `projects.json` at the workspace root
2. If only one project is registered: use it automatically
3. If multiple projects exist: ask the user to select one by name before proceeding

## Instructions

1. Follow the Story Prioritizer's Role, Rules, Trigger Conditions, and Execution Steps from `.claude/agents/story-prioritizer.md` exactly
2. Apply all 11 Global Rules from CLAUDE.md
3. Apply all Path Schema rules from CLAUDE.md
4. Check prerequisites: `context_approved: true` in `pipeline-state.json` (HALT if not approved)
5. Read parsed stories from `{PROJECT_OUTPUT}/stories/parsed/`
6. Read project context from `{PROJECT_OUTPUT}/context/project-context.md`
7. Score each story on Severity, Likelihood, DW (Dependency Weight), and Risk
8. Generate or update `{PROJECT_OUTPUT}/strategy/priority-matrix.md`
9. Archive previous version as `priority-matrix-v{N}.md` if updating an approved matrix
10. Present draft matrix for user approval via Gate 3
11. On approval: update `pipeline-state.json` with `strategy_approved: true` and increment `strategy_version`
12. Log all findings and assumptions via `assumption-tracker`

## Scoring Rules

- **Severity** — impact level if story behavior fails (Low=1, Medium=2, High=3, Critical=4)
  - Apply severity amplifier (+1) if story contains design references (Figma URLs, mockups, etc.)
  - Scan `figma_links`, `description`, AND all `sections[]` content for design URLs
- **Likelihood** — probability of discovering a defect during testing (1–4)
  - Filter out administrative comments (sprint assignments, PR links, status changes)
  - Only count risk signals: inconsistencies, ambiguities, missing specs
- **DW (Dependency Weight)** — whether other stories depend on this one
  - Set to 1 if any other story in the batch depends on this story
  - Set to 0 if no dependencies exist
  - Affects whether TC failure for this story blocks downstream testing
- **Risk** — Severity × Likelihood

## Extension Mode

When adding new stories to an already-approved matrix:
- Merge new story rows with existing rows
- Update DW on existing stories if new dependencies detected
- Keep all other columns (Severity, Likelihood, Risk Description, Rank) unchanged on existing rows
- Mark updated rows for confirmation in the approval gate

## Exception Handling

Follow the Exception Handling table in `.claude/agents/story-prioritizer.md` for:
- Context not approved (halt at Step 1)
- Null epic_key (score story without epic context)
- has_epics: false projects (never open epics/ folder)
- Dependency on missing epic (log assumption, use available context)
