# Project Permissions Setup

When you register a new project or want to eliminate permission prompts for an existing project, follow these steps to configure Claude Code permissions.

---

## Purpose

Permission allowlists pre-approve specific file and tool operations, so agents don't prompt you for permission on every read/write to registry files, story snapshots, test cases, etc.

---

## Setup Instructions

### Step 1: Locate Your Project Output Directory

Your project output path is defined in `projects.json`:

```json
{
  "project_name": "FormDesk-Aircraft (UI/UX)",
  "output_path": "C:/Users/.../Documents/FormDesk-Aircraft (UIUX)",
  ...
}
```

### Step 2: Copy the Settings Template

Copy this file:
```
PipelineQA/config/settings-clean.template.json
```

To your project's output directory as:
```
{PROJECT_OUTPUT}/.claude/settings.json
```

**Example:**
```
C:/Users/.../Documents/FormDesk-Aircraft (UIUX)/.claude/settings.json
```

### Step 3: (Optional) Customize

The template includes permissions for all standard agent directories. If your project uses a non-standard path or has custom folders, add entries for those too:

```json
{
  "permissions": {
    "allow": [
      "Read(custom-folder/*)",
      "Write(custom-folder/*)"
    ]
  }
}
```

### Step 4: Verify

Once the file is in place:
- Next time you run `/orchestrator`, you should see **fewer permission prompts**
- Agent reads/writes to registry, stories, test cases, etc. will be pre-approved

---

## What Gets Pre-Approved

| Operation | Example | Pre-Approved |
|-----------|---------|---|
| Registry updates | Write `pipeline-state.json` | ✅ Yes |
| Fetch stories | Read `stories/raw/*.raw.json` | ✅ Yes |
| Parse stories | Write `stories/parsed/*.parsed.json` | ✅ Yes |
| Generate TCs | Write `test-cases/*.csv` | ✅ Yes |
| Update strategy | Write `strategy/priority-matrix.md` | ✅ Yes |
| Track assumptions | Write `tracking/assumptions.md` | ✅ Yes |
| Jira API calls | `mcp__claude_ai_Atlassian_Rovo__getJiraIssue` | ✅ Yes |

---

## File Format

The `.claude/settings.json` file uses this structure:

```json
{
  "permissions": {
    "allow": [
      "Read(registry/pipeline-state.json)",
      "Write(registry/pipeline-state.json)",
      "Read(stories/**)",
      "Write(test-cases/*)",
      "mcp__claude_ai_Atlassian_Rovo__getJiraIssue"
    ]
  }
}
```

### Path Patterns

- `Read(file.md)` — exact file
- `Read(folder/*)` — all files in folder (not recursive)
- `Read(folder/**)` — all files recursively
- `Read(folder/**/*.ext)` — specific extension recursively

---

## Troubleshooting

### "Still seeing permission prompts"

1. Verify `.claude/settings.json` exists in the correct project output directory
2. Check that paths use `/` (forward slashes), not `\`
3. Reload the Claude Code session after adding the file

### "Want to add permissions later"

Just edit `.claude/settings.json` and add new entries to the `permissions.allow` array. No restart needed.

### "Want to remove pre-approval"

Delete the corresponding line from the `allow` array in `.claude/settings.json`.

---

## Template Files

Two templates are provided:

1. **`settings.template.json`** — Documented version with comments explaining each section
2. **`settings-clean.template.json`** — Clean JSON, ready to copy directly to `.claude/settings.json`

Use the clean version for fastest setup.

---

## Next Steps

1. Register your project (if new) via `/orchestrator`
2. Copy `settings-clean.template.json` → `{PROJECT_OUTPUT}/.claude/settings.json`
3. Run `/orchestrator` — you should see fewer prompts!
