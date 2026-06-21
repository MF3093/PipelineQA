# Orchestrator Agent Command

Read and execute the agent definition at `.github/agents/orchestrator.agent.md` exactly as written.

Apply the tool name mapping from CLAUDE.md:
- `read_file` → `Read` tool
- `grep_search` → `Grep` tool (NEVER on registry JSON files — use `Read` tool only)
- `file_search` → `Glob` tool
- `run_in_terminal` → `Bash` tool
- `edit_file` → `Edit` tool
- `create_file` → `Write` tool
- `tool_search` → `ToolSearch` tool

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
- Use the corresponding slash command instead: `/fetcher`, `/parser`, etc.
- Do NOT run them sequentially within a single response
- Instead: document what needs to be run, let the user invoke the agent, wait for completion, then continue the orchestration

## Instructions

1. Follow the Orchestrator's Role, Rules, Trigger Conditions, and Execution Steps from `.github/agents/orchestrator.agent.md` exactly
2. Apply all 10 Global Rules from CLAUDE.md
3. Apply all Path Schema rules from CLAUDE.md
4. Manage all approval gates using the protocol in Rule 9 of CLAUDE.md
5. Track pipeline state in `{PROJECT_OUTPUT}/registry/pipeline-state.json` according to the schema in the agent definition
6. At each phase boundary, present a gate (Fetch Gate, Context Gate, Strategy Gate, or Per-Story TC Gate) and wait for user response

## Approval Gate Protocol

Use ONLY these responses:
- Approve: `yes`, `y`, `approve`, `approved`, `aprobado`, `si`, `sí`, `ok`
- Edit: `edit`, `e`, `editar`, `change`, `cambiar`, `modify`, `modificar`
- Reject: `no`, `n`, `reject`, `rechazar`, `discard`, `cancel`

If the user's response is not recognized: re-prompt with "Response not recognized. Please reply with: yes / edit / reject"

## Exception Handling

Follow the Exception Handling table in `.github/agents/orchestrator.agent.md` for:
- Stale locks
- Missing pipeline state
- Corrupted JSON
- Registry inconsistencies
- Reconciliation failures
