# Parser Agent Command

Read and execute the agent definition at `.github/agents/parser.agent.md` exactly as written.

Apply the tool name mapping from CLAUDE.md:
- `read_file` → `Read` tool
- `grep_search` → `Grep` tool (NEVER on `fetched-stories.json`, `fetched-epics.json`, or `pipeline-state.json` — use `Read` tool only)
- `file_search` → `Glob` tool
- `run_in_terminal` → `Bash` tool
- `edit_file` → `Edit` tool
- `create_file` → `Write` tool
- `tool_search` → `ToolSearch` tool

## Inputs

The story key(s) to parse:
- If provided by the user (e.g., `/parser TEST-FMT-001`): use that key
- If no key provided: ask the user which story/stories to parse

To resolve `{PROJECT_OUTPUT}`:
1. Read `projects.json` at the workspace root
2. If only one project is registered: use it automatically
3. If multiple projects exist: ask the user to select one by name before proceeding

## Instructions

1. Follow the Parser's Role, Rules, Trigger Conditions, and Execution Steps from `.github/agents/parser.agent.md` exactly
2. Apply all 11 Global Rules from CLAUDE.md
3. Apply all Path Schema rules from CLAUDE.md
4. Self-verify the ParsedStory JSON before saving (use the checklist in the agent definition)
5. Update the registry entries as specified in Step 6 of the agent definition
6. Report all results to the user with the Parsing Summary format shown in Phase 3

## Exception Handling

Follow the Exception Handling table in `.github/agents/parser.agent.md` for:
- Missing raw files
- Unidentifiable AC sections
- Section header ambiguity
- Epic reference issues
- Pre-existing parsed files
- Resolved assumptions
