# Context Builder Agent Command

Read and execute the agent definition at `.github/agents/context-builder.agent.md` exactly as written.

Apply the tool name mapping from CLAUDE.md:
- `read_file` → `Read` tool
- `grep_search` → `Grep` tool (NEVER on `fetched-stories.json`, `fetched-epics.json`, or `pipeline-state.json` — use `Read` tool only)
- `file_search` → `Glob` tool
- `run_in_terminal` → `Bash` tool
- `edit_file` → `Edit` tool
- `create_file` → `Write` tool

## Inputs

Optional action:
- If the user says "Update project context": check if `project-context.md` is already approved and present the re-run confirmation gate first
- If invoked normally: proceed to interview

To resolve `{PROJECT_OUTPUT}`:
1. Read `projects.json` at the workspace root
2. If only one project is registered: use it automatically
3. If multiple projects exist: ask the user to select one by name before proceeding

## Instructions

1. Follow the Context Builder's Role, Rules, Trigger Conditions, and Execution Steps from `.github/agents/context-builder.agent.md` exactly
2. Apply all 10 Global Rules from CLAUDE.md
3. Apply all Path Schema rules from CLAUDE.md
4. Read parsed stories from `{PROJECT_OUTPUT}/stories/parsed/` to identify project signals
5. Scan story `sections[]` for non-obvious headers (e.g., "Backend Notes", "Testing Notes") that contain tech stack signals
6. Interview the user to capture:
   - Project overview and goals
   - Technology stack (language, frameworks, libraries)
   - QA environment (staging URL, test management tool)
   - Key stakeholders and conventions
7. Filter out story-specific content (story keys, AC references, sprint data, story-specific URLs, design tokens)
8. Present draft `project-context.md` for user approval
9. Save approved context to `{PROJECT_OUTPUT}/context/project-context.md`
10. Update `pipeline-state.json` with `context_approved: true` and `context_version: 1`
11. Log any unknowns via `assumption-tracker`

## Content Filtering Rules

- **Exclude**: Story keys, AC references, route URLs for specific stories, sprint data, design tokens, story-specific comments
- **Include**: Technology stack, QA infrastructure, cross-project patterns, conventions, glossary items that apply broadly

## Interview Fields

- Project overview
- Stakeholders & roles
- Technology stack (backend, frontend, databases, third-party APIs)
- QA environment (staging URL, test instance)
- Test management tool
- Key testing conventions or constraints
- Known risks or problem areas

## Exception Handling

Follow the Exception Handling table in `.github/agents/context-builder.agent.md` for:
- Re-run of approved context (present confirmation gate first)
- "Unknown" answers to interview questions (log as assumptions, use TBD placeholders)
- Missing parsed stories (build context from available stories)
- has_epics: false projects (build context from stories alone, never open epics/ folder)
