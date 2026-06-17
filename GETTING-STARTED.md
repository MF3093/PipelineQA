# Getting Started with PipelineQA

This guide walks you through setting up and running the QA pipeline for the first time.

---

## Prerequisites

| Requirement | Notes |
|---|---|
| VS Code | Latest stable version |
| GitHub Copilot Chat extension | Must be installed and signed in |
| Jira MCP server | Required only if your stories are in Jira. See [MCP setup](#mcp-setup-jira) below |

---

## Step 1 — Open the Workspace

Open the `PipelineQA` folder in VS Code:

```
File → Open Folder → select the PipelineQA folder
```

---

## Step 2 — MCP Setup (Jira)

> Skip this step if your stories are in Azure DevOps or you are not using Jira.

The Jira MCP server is already configured in `.vscode/mcp.json`. You only need to authenticate:

1. Open the Copilot Chat panel
2. Click the **MCP** icon or open the Command Palette → `MCP: List Servers`
3. Sign in to Atlassian when prompted
4. Confirm `MCPJira` shows as **connected**

---

## Step 3 — Register Your Project

Open Copilot Chat, select the `@orchestrator` agent, and type:

```
Start a new project
```

The Orchestrator will ask you for:

| Field | Example |
|---|---|
| Project name | `My App QA` |
| Output directory (absolute path) | `C:/Users/YourName/Documents/MyAppQA` |
| Bug tracking platform | `jira` / `ado` / `none` |
| Project key | `MYPROJ` (or `none`) |

Once registered, the Orchestrator creates the full output folder structure and copies the source configuration template.

---

## Step 4 — Fill In the Source Configuration

Open the generated file at:

```
{YOUR_OUTPUT_PATH}/config/source-config.md
```

Set the `story_source` field to your platform (`jira` or `ado`) and fill in the connection fields for that platform only. Leave the other platform section blank.

**Jira example:**
```
story_source: jira
Project Key:  MYPROJ
Cloud ID:     xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
Site:         yourcompany.atlassian.net
```

**Azure DevOps example:**
```
story_source: ado
Organization: your-org
Project Name: YourProject
```

---

## Step 5 — Run the Pipeline

In Copilot Chat with `@orchestrator`, tell the agent which stories to process:

```
Run the pipeline for stories MYPROJ-101, MYPROJ-102
```

The pipeline runs in two phases:

### Phase 1 — Early Analysis
`Fetch` → `Parse` → `Story Analyze`

The Orchestrator fetches the stories, parses them into a structured format, and analyzes them for ambiguities and coverage gaps.

### Phase 2 — TC Generation
`Context Build` → `Story Prioritizer` → `TC Generate`

On the **first run**, the Context Builder will conduct a short interview to capture your project's tech stack, test environment, and conventions. This only happens once.

---

## Approval Gates

The pipeline pauses at three gates for your review. Respond with:

| Intent | Accepted responses |
|---|---|
| Approve | `yes`, `y`, `ok`, `approve`, `si` |
| Request edits | `edit`, `change`, `modify` |
| Reject | `no`, `reject`, `cancel` |

| Gate | What you review |
|---|---|
| **Gate 2** | Project context (`project-context.md`) |
| **Gate 3** | Priority matrix (`priority-matrix.md`) |
| **Gate 4** | Test cases per story (one approval per story) |

---

## Output Structure

All artifacts are saved inside the output directory you configured:

```
{YOUR_OUTPUT_PATH}/
├── config/
│   └── source-config.md          ← Your connection config (fill this in Step 4)
├── context/
│   └── project-context.md        ← Approved project context (Gate 2)
├── stories/
│   ├── raw/                       ← Raw snapshots from Jira/ADO
│   └── parsed/                    ← Normalized ParsedStory JSON files
├── strategy/
│   └── priority-matrix.md         ← Approved risk-based priority matrix (Gate 3)
├── test-cases/                    ← Final test case files (one per story)
├── tracking/
│   ├── assumptions.md             ← Logged assumptions, questions, blockers
│   ├── corrections-log.md         ← Gate edits history
│   └── logs/                      ← Run logs
└── registry/
    ├── pipeline-state.json        ← Current pipeline state
    ├── fetched-stories.json       ← Story registry
    └── fetched-epics.json         ← Epic registry
```

---

## Running Subsequent Batches

For new stories on an already-registered project:

```
Run the pipeline for stories MYPROJ-110, MYPROJ-111
```

The Context Builder is **skipped** — it reuses the approved `project-context.md`. Only new, unprocessed stories are picked up.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| `MCPJira` not connected | Re-authenticate via `MCP: List Servers` in the Command Palette |
| Orchestrator says project not found | Run `Start a new project` to register it |
| Gate response not recognized | Use only the accepted values listed above — no punctuation |
| Agent skips a story | Check `registry/fetched-stories.json` — the story may already be marked as approved |
