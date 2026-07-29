# Orchestrator Agent Command

Read and execute the agent definition at `.claude/agents/orchestrator.md` exactly as written.

Apply Claude Code tool standards (Read, Edit, Grep, Bash, etc.).

## Inputs

Optional action specifier:
- If the user provides a specific phase or action (e.g., `/orchestrator phase 1` or `/orchestrator tc generation`): follow that directly
- If no input provided: present the run menu (Step 8 in the agent definition) and wait for user input

To resolve `{PROJECT_OUTPUT}`:
1. Read `projects.json` at the workspace root
2. At Step 1 (Project Selection): list all registered projects and ask the user to select one
3. Once selected, use that project's `output_path` as `{PROJECT_OUTPUT}` for all subsequent steps

## Critical Notes for Claude Code

The agent definition references "subagent" invocations (e.g., "invoke the Fetcher as a subagent"). In Claude Code:
- Invoke the corresponding agent directly via the Agent tool (subagent_type: fetcher, parser, story-analyzer, context-builder, story-prioritizer, tc-generator, tc-reviewer) — do NOT ask the user to run the `/fetcher`, `/parser`, etc. slash commands themselves.
- This matches the platform-difference note in `.claude/agents/orchestrator.md`'s frontmatter: Claude Code can invoke subagents directly and load deferred tools inside them, unlike the Copilot source, so no manual hand-off workaround is needed.
- Wait for each subagent's result before proceeding to the next phase or gate, exactly as the Full Pipeline Sequence table in the agent definition specifies.

## Instructions

1. Follow the Orchestrator's Role, Rules, Trigger Conditions, and Execution Steps from `.claude/agents/orchestrator.md` exactly
2. Apply all 11 Global Rules from `.claude/instructions/global-rules.md`
3. Apply all Path Schema rules from `.claude/instructions/path-schema.md`
4. Manage all approval gates using the protocol in Rule 9 of Global Rules
5. Track pipeline state in `{PROJECT_OUTPUT}/registry/pipeline-state.json` according to the schema in the agent definition
6. At each phase boundary, present a gate (Fetch Gate, Context Gate, Strategy Gate, or Per-Story TC Gate) and wait for user response

## Approval Gate Protocol

Use ONLY these responses:
- Approve: `yes`, `y`, `approve`, `approved`, `aprobado`, `si`, `sí`, `ok`
- Edit: `edit`, `e`, `editar`, `change`, `cambiar`, `modify`, `modificar`
- Reject: `no`, `n`, `reject`, `rechazar`, `discard`, `cancel`

If the user's response is not recognized: re-prompt with "Response not recognized. Please reply with: yes / edit / reject"

## Exception Handling

Follow the Exception Handling table in `.claude/agents/orchestrator.md` for:
- Stale locks
- Missing pipeline state
- Corrupted JSON
- Registry inconsistencies
- Reconciliation failures
