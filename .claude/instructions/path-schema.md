# Path Schema — QA Pipeline Standard Folder & File Structure

All QA artifacts for a project are stored in `{PROJECT_OUTPUT}` as defined in `projects.json`.

## Critical Path Rules

| Rule | Constraint |
|------|-----------|
| **P-1** | Pipeline state ONLY at `{PROJECT_OUTPUT}/registry/pipeline-state.json` |
| **P-2** | Test cases flat: `test-cases/{STORY-KEY}-test-cases.csv` (no subfolders) |
| **P-3** | **If `has_screenshots: false`:** `screenshots/` folder does not exist. **If `has_screenshots: true` AND `has_epics: true`:** `screenshots/{EPIC-KEY}/`; **If `has_screenshots: true` AND `has_epics: false`:** `screenshots/{STORY-KEY}/` |
| **P-4** | **If `has_epics: false`:** `epics/` folder does not exist. **If `has_epics: true`:** `epics/raw/` and `epics/parsed/` folders exist. |
| **P-5** | **If `has_extra_resources: false`:** `ExtraResources/` folder does not exist. **If `has_extra_resources: true`:** folder structure depends on epic flag. |
| **P-6** | Parsed stories: `stories/parsed/{STORY-KEY}.parsed.json` |
| **P-7** | Test data centralized: `test-cases/test-data-requirements.md` (single file, not per-story) |
| **P-8** | TC Reviewer reports: `tracking/reviews/cross-story-review-{batch_id}.md` or `integration-review-{batch_id}.md` |

## Folder Structure

### ✅ Always Present

```
{PROJECT_OUTPUT}/
├── registry/
│   ├── pipeline-state.json          ← ONLY location for pipeline state
│   ├── fetched-stories.json         ← Story metadata registry
│   └── fetched-epics.json           ← Epic metadata registry
├── stories/
│   ├── raw/{STORY-KEY}.raw.json     ← Raw API response per story
│   └── parsed/{STORY-KEY}.parsed.json ← Extracted AC + technical details
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
└── config/source-config.md          ← Copied from template at registration
```

### ⚙️ Conditional: Epics

**If `has_epics: true`:** Create these folders and populate them.
**If `has_epics: false`:** Do NOT create these folders; Parser and Story Prioritizer skip epic processing.

```
epics/
├── raw/{EPIC-KEY}.raw.json          ← Raw API response per epic
└── parsed/{EPIC-KEY}.parsed.json    ← Extracted goal, ACs, out-of-scope, known_story_keys
```

### ⚙️ Conditional: Screenshots

**If `has_screenshots: false`:** Do NOT create `screenshots/` folder; TC Generator skips all screenshot reads.
**If `has_screenshots: true` AND `has_epics: true`:** Create and populate epic-scoped screenshot folders.
**If `has_screenshots: true` AND `has_epics: false`:** Create and populate story-scoped screenshot folders.

```
screenshots/
├── {EPIC-KEY}/                      ← if `has_epics: true`
│   ├── {EPIC-KEY}-screenshot-001.png
│   └── {EPIC-KEY}-screenshot-NNN.png
└── {STORY-KEY}/                     ← if `has_epics: false`
    ├── {STORY-KEY}-screenshot-001.png
    └── {STORY-KEY}-screenshot-NNN.png
```

### ⚙️ Conditional: ExtraResources

**If `has_extra_resources: false`:** Do NOT create `ExtraResources/` folder; agents skip resource reads.
**If `has_extra_resources: true` AND `has_epics: true`:** Create root-level and epic-scoped resource folders.
**If `has_extra_resources: true` AND `has_epics: false`:** Create root-level and story-scoped resource folders.

```
ExtraResources/
├── {project-wide file}              ← Root-level: applies to ALL stories
├── {EPIC-KEY}/                      ← if `has_epics: true`
│   ├── {resource-file}
│   └── {resource-file}
└── {STORY-KEY}/                     ← if `has_epics: false`
    ├── {resource-file}
    └── {resource-file}
```

## Agent Path Assignments

| Agent | Reads | Writes | Notes |
|-------|-------|--------|-------|
| Fetcher | — | `stories/raw/`, `epics/raw/` (if `has_epics: true`) | Skips epic write if has_epics: false |
| Parser | `stories/raw/` | `stories/parsed/` | Always. `epics/parsed/` only if `has_epics: true` |
| Context Builder | `stories/parsed/` | `context/project-context.md` | — |
| Story Prioritizer | `stories/parsed/`, `context/` | `strategy/priority-matrix.md`, `strategy/strategy-versions/` | Reads `epics/parsed/` only if `has_epics: true` |
| TC Generator | `stories/parsed/` | `test-cases/` | Reads `screenshots/` only if `has_screenshots: true`; reads `ExtraResources/` only if `has_extra_resources: true` |
| Story Analyzer | `stories/parsed/` | (none) | Reads `screenshots/` only if `has_screenshots: true`; reads `epics/parsed/` only if `has_epics: true` |
| TC Reviewer | `test-cases/`, `strategy/`, `tracking/` | `tracking/reviews/` (optional, user-confirmed) | Read-only cross-story analysis |
| Orchestrator | `registry/` (all) | `registry/` (all) | Manages folder creation per flags |

## Initialization Checklist (at project registration)

The **Orchestrator** performs folder initialization during project registration. Conditionals are checked against `projects.json` flags.

### Always Create

**Folders:** `registry/`, `stories/raw/`, `stories/parsed/`, `context/`, `strategy/`, `strategy/strategy-versions/`, `test-cases/`, `tracking/`, `tracking/archive/`, `tracking/reviews/`, `config/`

### Create Conditionally

**If `has_epics: true`:**
- Create: `epics/raw/`, `epics/parsed/`

**If `has_epics: false`:**
- Do NOT create `epics/` folders

**If `has_screenshots: true`:**
- Create: `screenshots/`

**If `has_screenshots: false`:**
- Do NOT create `screenshots/` folder

**If `has_extra_resources: true`:**
- Create: `ExtraResources/`

**If `has_extra_resources: false`:**
- Do NOT create `ExtraResources/` folder

## Path Resolution Examples

- Screenshot: `{PROJECT_OUTPUT}/screenshots/H20-37/H20-37-screenshot-001.png`
- TC file: `{PROJECT_OUTPUT}/test-cases/H20-52-test-cases.csv`
- Pipeline state: `{PROJECT_OUTPUT}/registry/pipeline-state.json`
- Parsed story: `{PROJECT_OUTPUT}/stories/parsed/H20-52.parsed.json`
