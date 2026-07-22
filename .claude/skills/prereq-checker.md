# Skill: prereq-checker

## Invocation

```
prereq-checker.invoke({
  agent: string,                  // name of the calling agent (for error messages)
  checks: [
    {
      type: "file_exists",        // verify a file exists at the given path
      path: string,               // absolute or {PROJECT_OUTPUT}-relative path
      label: string               // human-readable name for error messages
    } |
    {
      type: "registry_status",    // verify a story has a specific status in the registry
      registry: "stories" | "epics",
      key: string,                // story key or epic key
      expected_status: string,    // e.g. "parsed", "fetched", "approved"
      label: string
    } |
    {
      type: "pipeline_flag",      // verify a boolean flag in pipeline-state.json
      flag: string,               // e.g. "context_approved", "strategy_approved"
      expected: boolean,
      label: string
    } |
    {
      type: "field_not_tbd",      // verify a field in project-context.md is not [TBD]
      field_name: string,         // exact field name as it appears in project-context.md
      label: string
    }
  ]
}) → {
  passed: boolean,
  failures: [
    {
      type: string,
      label: string,
      path_or_key: string,
      message: string,            // human-readable explanation
      suggestion: string          // what the user should do to resolve it
    }
  ],
  field_values: { [field_name: string]: string }  // populated only for `field_not_tbd` checks that PASSED — the actual field value read during validation
}
```

---

## Behavior

1. Run ALL checks — do not stop at first failure.
2. If `passed: false`: calling agent stops, presents failures:
```
"Prerequisites check failed — {agent name} cannot start.
{N} issue(s) found:
1. [{label}] Issue: {message} Fix: {suggestion}
Resolve the above and retry."
```
3. If `passed: true`: silent pass — do not report to user.
4. For every `field_not_tbd` check that passes, return the field's actual value in `field_values[field_name]`. The calling agent MUST reuse this value instead of re-reading the source file to fetch the same field — this avoids a redundant full-file read immediately after prereq-checker already opened it for validation.

## Check Rules

| Type | Pass condition | Failure message |
|------|---------------|-----------------|
| `file_exists` | File exists at `path` | `"File not found: {path}"` |
| `registry_status` | Entry `key` has `status = expected_status` | `"Story/Epic {key} has status '{actual}', expected '{expected}'"` |
| `pipeline_flag` | `approval_states.{flag} = expected` | `"Pipeline flag '{flag}' is {actual}, expected {expected}"` |
| `field_not_tbd` | Field in `project-context.md` does not contain `[TBD` | `"Field '{field_name}' in project-context.md is still [TBD]"` |

---

## Standard Prerequisite Sets

**Quick reference** — agents call `prereq-checker.invoke({ agent: "{name}", checks: [...] })` using their named profile below:

| Agent | Profile name to use | Checks |
|-------|--------------------|-|
| Fetcher | Fetcher | projects.json + story registry + epic registry |
| Parser | Parser | story registry + one `registry_status` per story in batch |
| Context Builder | Context Builder | pipeline-state.json |
| Prioritizer | Prioritizer | context_approved flag + one parsed story file per batch story |
| Story Analyzer | Story Analyzer | parsed story file + story registry status + assumptions.md |
| TC Generator | TC Generator | context_approved + strategy_approved + parsed story + 2 TBD field checks |
| TC Reviewer | TC Reviewer | projects.json + priority-matrix.md + assumptions.md |
| Bug Reporter | Bug Reporter | projects.json + story registry + epic registry (+ TC file in Mode A) |

Full check definitions for each profile:

### Fetcher
```
checks: [
  { type: "file_exists", path: "projects.json", label: "Project registry" },
  { type: "file_exists", path: "{PROJECT_OUTPUT}/registry/fetched-stories.json", label: "Story registry" },
  { type: "file_exists", path: "{PROJECT_OUTPUT}/registry/fetched-epics.json", label: "Epic registry" }
]
```

### Parser
```
checks: [
  { type: "file_exists", path: "{PROJECT_OUTPUT}/registry/fetched-stories.json", label: "Story registry" },
  // one per story in batch:
  { type: "registry_status", registry: "stories", key: "{STORY-KEY}", expected_status: "fetched", label: "Story {STORY-KEY} ready to parse" }
]
```

### Context Builder
```
checks: [
  { type: "file_exists", path: "{PROJECT_OUTPUT}/registry/pipeline-state.json", label: "Pipeline state" }
]
```

### Prioritizer
```
checks: [
  { type: "pipeline_flag", flag: "context_approved", expected: true, label: "Project context approved" },
  // one per story in batch:
  { type: "file_exists", path: "{PROJECT_OUTPUT}/stories/parsed/{STORY-KEY}.parsed.json", label: "Parsed story {STORY-KEY}" }
]
```

### Story Analyzer
```
checks: [
  // one per story being analyzed:
  { type: "file_exists", path: "{PROJECT_OUTPUT}/stories/parsed/{STORY-KEY}.parsed.json", label: "Parsed story {STORY-KEY}" },
  { type: "registry_status", registry: "stories", key: "{STORY-KEY}", expected_status: "parsed", label: "Story {STORY-KEY} ready for analysis" },
  { type: "file_exists", path: "{PROJECT_OUTPUT}/tracking/assumptions.md", label: "Assumptions log" }
]
```

### TC Generator
```
checks: [
  { type: "pipeline_flag", flag: "context_approved", expected: true, label: "Project context approved" },
  { type: "pipeline_flag", flag: "strategy_approved", expected: true, label: "Test strategy approved" },
  { type: "file_exists", path: "{PROJECT_OUTPUT}/stories/parsed/{STORY-KEY}.parsed.json", label: "Parsed story {STORY-KEY}" },
  { type: "field_not_tbd", field_name: "Test Management Tool", label: "Test Management Tool defined" },
  { type: "field_not_tbd", field_name: "Import Format", label: "Import Format defined" }
]
```

### TC Reviewer
```
checks: [
  { type: "file_exists", path: "projects.json", label: "Project registry" },
  { type: "file_exists", path: "{PROJECT_OUTPUT}/strategy/priority-matrix.md", label: "Approved priority matrix" },
  { type: "file_exists", path: "{PROJECT_OUTPUT}/tracking/assumptions.md", label: "Assumptions log" }
]
// Additional runtime check (not via prereq-checker): ≥ 2 TC files must exist in test-cases/
```

### Bug Reporter
```
checks: [
  { type: "file_exists", path: "projects.json", label: "Project registry" },
  { type: "file_exists", path: "{PROJECT_OUTPUT}/registry/fetched-stories.json", label: "Story registry" },
  { type: "file_exists", path: "{PROJECT_OUTPUT}/registry/fetched-epics.json", label: "Epic registry" }
]
// Mode A only — after story key is resolved from TC ID:
// { type: "file_exists", path: "{PROJECT_OUTPUT}/test-cases/{STORY-KEY}-test-cases.csv", label: "Test case file for {STORY-KEY}" }
```

---

## Permissions
- Read: `projects.json`, `{PROJECT_OUTPUT}/registry/`, `{PROJECT_OUTPUT}/context/project-context.md`
- Cannot write any file.
- Cannot invoke any other skill or agent.
