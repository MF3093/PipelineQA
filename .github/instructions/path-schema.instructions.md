---
description: "Use when writing files to the project output folder or resolving file paths. Defines the canonical folder structure and critical path rules."
applyTo: ".github/agents/**"
---

# Path Schema — QA Pipeline Standard Folder & File Structure

All QA artifacts for a project are stored in `{PROJECT_OUTPUT}` as defined in `projects.json`.

## Critical Path Rules

| Rule | Constraint |
|------|-----------|
| **P-1** | Pipeline state ONLY at `{PROJECT_OUTPUT}/registry/pipeline-state.json` |
| **P-2** | Test cases flat: `test-cases/{STORY-KEY}-test-cases.csv` (no subfolders) |
| **P-3** | Screenshots subfolder: if `has_epics: true` → `screenshots/{EPIC-KEY}/`; if `has_epics: false` → `screenshots/{STORY-KEY}/` |
| **P-4** | Parsed stories: `stories/parsed/{STORY-KEY}.parsed.json` |
| **P-5** | Test data centralized: `test-cases/test-data-requirements.md` (single file, not per-story) |
| **P-6** | TC Reviewer reports: `tracking/reviews/cross-story-review-{batch_id}.md` or `integration-review-{batch_id}.md` |

## Folder Structure

```
{PROJECT_OUTPUT}/
├── registry/
│   ├── pipeline-state.json          ← ONLY location for pipeline state
│   ├── fetched-stories.json         ← Story metadata registry
│   └── fetched-epics.json           ← Epic metadata registry
├── stories/
│   ├── raw/{STORY-KEY}.raw.json     ← Raw API response per story
│   └── parsed/{STORY-KEY}.parsed.json ← Extracted AC + technical details
├── epics/                           ← Optional — only if `has_epics: true`
│   ├── raw/{EPIC-KEY}.raw.json
│   └── parsed/{EPIC-KEY}.parsed.json
├── context/
│   └── project-context.md           ← Project-wide context (tech stack, conventions)
├── strategy/
│   ├── priority-matrix.md           ← Current approved priority matrix
│   └── strategy-versions/           ← Archived versions (priority-matrix-v1.md, etc.)
├── test-cases/
│   ├── {STORY-KEY}-test-cases.csv   ← One TC file per story (flat, NO subfolders)
│   └── test-data-requirements.md    ← Centralized test data (TD-NNN entries)
├── tracking/
│   ├── assumptions.md               ← Open/resolved assumptions, questions, blockers
│   ├── corrections-log.md           ← Manual corrections (COR-NNN entries)
│   ├── archive/                     ← Archived assumption batches
│   ├── logs/{RUN-ID}.log.md         ← One log per run
│   └── reviews/                     ← TC Reviewer cross-story/integration reports
├── screenshots/                     ← Optional — only if `has_screenshots: true`
│   ├── {EPIC-KEY}/                    ← if `has_epics: true`
│   └── {STORY-KEY}/                   ← if `has_epics: false`
├── config/source-config.md          ← Copied from template at registration
└── ExtraResources/                  ← Optional — only if `has_extra_resources: true`
    ├── {project-wide file}            ← Root-level files apply to ALL stories (e.g. full prototype report)
    ├── {EPIC-KEY}/                    ← if `has_epics: true` — scoped to that epic
    └── {STORY-KEY}/                   ← if `has_epics: false` — scoped to that story
```

## Agent Path Assignments

| Agent | Reads | Writes |
|-------|-------|--------|
| Fetcher | — | `stories/raw/`, `epics/raw/` |
| Parser | `stories/raw/` | `stories/parsed/` |
| Context Builder | `stories/parsed/` | `context/project-context.md` |
| Strategy | `stories/parsed/`, `context/` | `strategy/priority-matrix.md`, `strategy/strategy-versions/` |
| TC Generator | `stories/parsed/`, `screenshots/` | `test-cases/` |
| TC Reviewer | `test-cases/`, `strategy/`, `tracking/` | `tracking/reviews/` (optional, user-confirmed) |
| Orchestrator | `registry/` (all) | `registry/` (all) |

## Initialization Checklist (at project registration)

**Folders (always):** registry/, stories/raw/, stories/parsed/, context/, strategy/strategy-versions/, test-cases/, tracking/archive/, tracking/reviews/, config/

**Folders (conditional):** epics/raw/, epics/parsed/ — only if `has_epics: true`; screenshots/ — only if `has_screenshots: true`; ExtraResources/ — only if `has_extra_resources: true`

**Files:**
- `registry/pipeline-state.json` → initialized with schema from orchestrator
- `registry/fetched-stories.json` → `{ "schema_version": "1.0", "last_updated": "", "stories": [] }`
- `registry/fetched-epics.json` → `{ "schema_version": "1.0", "last_updated": "", "epics": [] }`

## Path Resolution Examples

- Screenshot: `{PROJECT_OUTPUT}/screenshots/H20-37/H20-37-screenshot-001.png`
- TC file: `{PROJECT_OUTPUT}/test-cases/H20-52-test-cases.csv`
- Pipeline state: `{PROJECT_OUTPUT}/registry/pipeline-state.json`
- Parsed story: `{PROJECT_OUTPUT}/stories/parsed/H20-52.parsed.json`
