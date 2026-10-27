# Source Configuration Template

> This is the shared template. Project-specific configuration lives in
> `{PROJECT_OUTPUT}/config/source-config.md` for each registered project.
> When registering a new project, copy this template to the project output folder
> and fill in the connection values below.

---

## Platform
- **story_source:** jira

---

## Jira

### Connection
- **MCP Tool:** `mcp__claude_ai_Atlassian_Rovo__getJiraIssue`
- **Permission:** Read-only — never create, edit, comment on, transition, or modify any Jira issue

### Project
- **Project Name:** {PROJECT_NAME}
- **Project Key:** {PROJECT_KEY}
- **Cloud ID:** {CLOUD_ID}
- **Site:** {SITE}

### Fields to Request
```
["key", "summary", "issuetype", "priority", "status", "description", "parent", "comment", "issuelinks"]
```

### Epic Fields to Request
```
["key", "summary", "issuetype", "priority", "status", "description"]
```

### Query
Fetch each specified story key individually by ID — no JQL needed.

### Parent Entity Type
`Epic`
