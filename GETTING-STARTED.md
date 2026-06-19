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

`.vscode/mcp.json` is excluded from version control (it may contain personal tokens). You must create it manually:

1. Create the file `.vscode/mcp.json` in the workspace root with the following content:

```json
{
  "servers": {
    "MCPJira": {
      "type": "http",
      "url": "https://mcp.atlassian.com/v1/mcp"
    }
  }
}
```

2. Open the Copilot Chat panel
3. Click the **MCP** icon or open the Command Palette → `MCP: List Servers`
4. Sign in to Atlassian when prompted
5. Confirm `MCPJira` shows as **connected**

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
| Project key | `MYPROJ` (or `none`) |
| Stories grouped under epics? | `yes` / `no` |
| UI screenshots available? | `yes` / `no` |
| Supplementary documentation? | `yes` / `no` |

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

In Copilot Chat with `@orchestrator`, start a run:

```
@orchestrator I want to run the pipeline
```

The Orchestrator will:
1. List your registered projects — select one by number
2. Present the run menu — choose an option:

| Option | What it runs |
|---|---|
| `1` | Full run — all phases |
| `2` | Phase 1 only (Fetch → Parse → Story Analysis) |
| `3` | Phase 2 only (Story Prioritizer → TC Generation) |
| `4`–`7` | Individual phases |
| `8` | Update project context |

3. Provide the story IDs when prompted:

```
MYPROJ-101, MYPROJ-102
```

The pipeline runs in two phases:

### Phase 1 — Early Analysis
`Fetch` → `Parse` → `Story Analyze`

The Orchestrator fetches the stories, parses them into a structured format, and analyzes them for ambiguities and coverage gaps.

### First Run Only — Project Context
`Context Build` → **Gate 2**

Before TC generation can begin, the Context Builder conducts a short interview to capture your project's tech stack, test environment, and conventions. The output (`project-context.md`) must be approved at Gate 2. This step is skipped on all subsequent runs.

### Phase 2 — TC Generation
`Story Prioritizer` → `TC Generate`

Generates a risk-based priority matrix (Gate 3) and then produces test cases per story (Gate 4).

---

## Step 6 — Add Screenshots and Extra Resources (Optional)

If you answered **yes** to UI screenshots or supplementary documentation during project registration, the pipeline creates folders for them. The folder structure depends on whether your stories are grouped under epics.

### Adding Screenshots

Screenshots go in the `screenshots/` folder. Structure them based on your project layout:

**If stories are grouped under epics:**
```
{YOUR_OUTPUT_PATH}/screenshots/
├── EPIC-1/
│   ├── MYPROJ-101_login_screen.png
│   ├── MYPROJ-101_error_state.png
│   └── MYPROJ-102_dashboard_overview.png
└── EPIC-2/
    ├── MYPROJ-103_settings_panel.png
    └── MYPROJ-104_export_dialog.png
```

**If stories are NOT grouped under epics (flat structure):**
```
{YOUR_OUTPUT_PATH}/screenshots/
├── MYPROJ-101_login_screen.png
├── MYPROJ-101_error_state.png
├── MYPROJ-102_dashboard_overview.png
└── MYPROJ-103_settings_panel.png
```

**How to add them:**

1. Create subfolders matching your epic names (if using epics), or add directly to `screenshots/`
2. Save UI screenshots in PNG, JPG, or WebP format
3. Use descriptive filenames tied to story IDs:
   ```
   MYPROJ-101_login_screen.png
   MYPROJ-102_error_handling.png
   MYPROJ-103_validation_messages.png
   ```
4. The Orchestrator will reference these during Phase 1 analysis and TC generation

### Adding Extra Resources / Documentation

Extra resources (requirements docs, design specs, API docs, etc.) go in the `ExtraResources/` folder, organized the same way:

**If stories are grouped under epics:**
```
{YOUR_OUTPUT_PATH}/ExtraResources/
├── EPIC-1/
│   ├── API-specification.md
│   ├── Design-guidelines.pdf
│   └── Database-schema.json
└── EPIC-2/
    ├── Acceptance-criteria.docx
    └── Integration-notes.md
```

**If stories are NOT grouped under epics (flat structure):**
```
{YOUR_OUTPUT_PATH}/ExtraResources/
├── API-specification.md
├── Design-guidelines.pdf
├── Database-schema.json
├── Acceptance-criteria.docx
└── Integration-notes.md
```

**How to add them:**

1. Create subfolders matching your epic names (if using epics), or add directly to `ExtraResources/`
2. Copy supplementary documentation files (Markdown, PDF, Word, text, or JSON)
3. Use clear, descriptive filenames:
   ```
   API-specification.md
   Design-guidelines.pdf
   Database-schema.json
   Acceptance-criteria.docx
   ```
4. Reference them in your Jira/ADO story descriptions if relevant:
   ```
   See ExtraResources/API-specification.md for endpoint details
   See ExtraResources/EPIC-1/Design-guidelines.pdf for UI standards
   ```

**Timing:**
- Add screenshots and resources **before** starting a pipeline run for best results
- The Orchestrator uses them during story analysis and TC generation to understand the full context
- You can add them between phases if needed, but they won't be picked up until the next run

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
├── epics/                         ← Only created if stories have epics
├── strategy/
│   └── priority-matrix.md         ← Approved risk-based priority matrix (Gate 3)
├── test-cases/                    ← Final test case files (one per story)
├── tracking/
│   ├── assumptions.md             ← Logged assumptions, questions, blockers
├── screenshots/                   ← Only created if you answered yes to screenshots
├── ExtraResources/                ← Only created if you answered yes to extra docs
│   └── corrections-log.md         ← Gate edits history
└── registry/
    ├── pipeline-state.json        ← Current pipeline state
    ├── fetched-stories.json       ← Story registry
    └── fetched-epics.json         ← Only created if stories have epics
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
