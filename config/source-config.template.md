# Source Configuration Template

> This is the shared template. Project-specific configuration lives in
> `{PROJECT_OUTPUT}/config/source-config.md` for each registered project.
> When registering a new project, copy this template to the project output folder,
> set `story_source` to the active platform, and fill in the connection values for that platform only.
> The Fetcher reads `## Active Platform` first, then the matching platform section.

---

## Active Platform
- **story_source:** {jira | ado}

---

## Jira

### Connection
- **MCP Tool:** `mcp_jiramcp_*`
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

---

## Azure DevOps

### Connection
- **MCP Tool:** `mcp_ado_*`
- **Permission:** Read-only — never create, edit, comment on, transition, or modify any work item

### Project
- **Organization:** {ADO_ORGANIZATION}
- **Project Name:** {ADO_PROJECT_NAME}

### Fields to Request
```
["System.Id", "System.Title", "System.WorkItemType", "Microsoft.VSTS.Common.Priority",
 "System.State", "System.Description", "System.Parent", "System.AreaPath"]
```

> **Note:** `relations` are returned automatically in ADO work item responses (via `$expand=relations` or included by default). The Parser uses `payload.relations[]` to extract linked stories.

### Epic Fields to Request
```
["System.Id", "System.Title", "System.WorkItemType", "Microsoft.VSTS.Common.Priority",
 "System.State", "System.Description"]
```

### Query
Fetch each specified work item ID individually — no WIQL needed.

### Parent Entity Type
`Feature` (or `Epic` depending on the process template)

