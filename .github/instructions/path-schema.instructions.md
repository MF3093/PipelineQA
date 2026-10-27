---
description: "Use when writing files to the project output folder or resolving file paths. Defines the canonical folder structure and critical path rules."
applyTo: ".github/agents/**"
---

# Path Schema — QA Pipeline Standard Folder & File Structure

All QA artifacts for a project are stored in `{PROJECT_OUTPUT}` as defined in `projects.json`.

## Critical Path Rules

| Rule | Constraint |
|------|-----------|
| **P-1** | Pipeline state ONLY at `registry/pipeline-state.json` |
| **P-2** | Test cases flat: `test-cases/{STORY-KEY}-test-cases.csv` (no subfolders) |
| **P-3** | Screenshots: folder exists only if `has_screenshots: true`; scoped by `{EPIC-KEY}/` if `has_epics: true`, else `{STORY-KEY}/` |
| **P-4** | Epics: `epics/raw/` + `epics/parsed/` exist only if `has_epics: true` |
| **P-5** | ExtraResources: folder exists only if `has_extra_resources: true`; scoped same as screenshots |
| **P-6** | Parsed stories: `stories/parsed/{STORY-KEY}.parsed.json` |
| **P-7** | Test data centralized: `test-cases/test-data-requirements.md` (single file, not per-story) |
| **P-8** | TC Reviewer reports: `tracking/reviews/cross-story-review-{batch_id}.md` or `integration-review-{batch_id}.md` |

## Folder Structure

### Always Present

```
{PROJECT_OUTPUT}/
├── registry/
│   ├── pipeline-state.json
│   ├── fetched-stories.json
│   └── fetched-epics.json
├── stories/
│   ├── raw/{STORY-KEY}.raw.json
│   └── parsed/{STORY-KEY}.parsed.json
├── context/
│   └── project-context.md
├── strategy/
│   ├── priority-matrix.md
│   └── strategy-versions/
├── test-cases/
│   ├── {STORY-KEY}-test-cases.csv
│   └── test-data-requirements.md
├── tracking/
│   ├── assumptions.md
│   ├── corrections-log.md
│   ├── archive/
│   ├── logs/{RUN-ID}.log.md
│   └── reviews/
├── cache/
│   ├── extra-resources-summary.md
│   ├── screenshot-reference-{SCOPE-KEY}.md
│   ├── priority-plan-{batch_id}.json      ← scratch; deleted after approval
│   └── tc-plan-{batch_id}.json            ← scratch; deleted after last story
└── config/source-config.md
```

### Conditional Folders

| Flag | `true` | `false` |
|------|--------|---------|
| `has_epics` | Create `epics/raw/`, `epics/parsed/` | No `epics/` folder; skip epic processing |
| `has_screenshots` | Create `screenshots/` with `{EPIC-KEY}/` or `{STORY-KEY}/` subfolders per epic flag | No `screenshots/` folder; skip screenshot reads |
| `has_extra_resources` | Create `ExtraResources/` with root-level files + `{EPIC-KEY}/` or `{STORY-KEY}/` subfolders per epic flag | No `ExtraResources/` folder; skip resource reads |

## Agent Path Assignments

| Agent | Reads | Writes | Conditional |
|-------|-------|--------|-------------|
| Fetcher | — | `stories/raw/`, `tracking/logs/` | `epics/raw/` if has_epics |
| Parser | `stories/raw/` | `stories/parsed/`, `tracking/logs/` | `epics/parsed/` if has_epics |
| Context Builder | `stories/parsed/` | `context/project-context.md` | — |
| Story Analyzer | `stories/parsed/` | `cache/extra-resources-summary.md`, `cache/screenshot-reference-*.md` | `screenshots/`, `ExtraResources/`, `epics/parsed/` per flags |
| Story Prioritizer | `stories/parsed/`, `context/` | `strategy/priority-matrix.md`, `strategy/strategy-versions/`, `cache/priority-plan-*.json` | `epics/parsed/` if has_epics |
| TC Generator | `stories/parsed/` | `test-cases/`, `cache/tc-plan-*.json` | `screenshots/`, `ExtraResources/`, `cache/` summaries per flags |
| TC Reviewer | `test-cases/`, `strategy/`, `tracking/` | `tracking/reviews/` | — |
| Orchestrator | `registry/` (all) | `registry/` (all) | Manages folder creation per flags |

## Initialization (Orchestrator at registration)

**Always create folders:** `registry/`, `stories/raw/`, `stories/parsed/`, `context/`, `strategy/`, `strategy/strategy-versions/`, `test-cases/`, `tracking/`, `tracking/archive/`, `tracking/reviews/`, `cache/`, `config/`

**Always create files:**
- `registry/pipeline-state.json` — initialized with orchestrator schema
- `registry/fetched-stories.json` — `{ "schema_version": "1.0", "last_updated": "", "stories": [] }`
- `registry/fetched-epics.json` — `{ "schema_version": "1.0", "last_updated": "", "epics": [] }`

**Conditional:** create folders per the Conditional Folders table above. Subfolders (`{EPIC-KEY}/` or `{STORY-KEY}/`) are created as stories/epics are fetched, not at registration.

## Path Resolution Examples

- Screenshot: `{PROJECT_OUTPUT}/screenshots/H20-37/H20-37-screenshot-001.png`
- TC file: `{PROJECT_OUTPUT}/test-cases/H20-52-test-cases.csv`
- Pipeline state: `{PROJECT_OUTPUT}/registry/pipeline-state.json`
- Parsed story: `{PROJECT_OUTPUT}/stories/parsed/H20-52.parsed.json`
