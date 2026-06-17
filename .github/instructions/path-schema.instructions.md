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
| **P-3** | Screenshots by epic: `screenshots/{EPIC-KEY}/` (NOT by story) |
| **P-4** | Parsed stories: `stories/parsed/{STORY-KEY}.parsed.json` |
| **P-5** | Test data centralized: `test-cases/test-data-requirements.md` (single file, not per-story) |
| **P-6** | TC Reviewer reports: `tracking/reviews/cross-story-review-{batch_id}.md` or `integration-review-{batch_id}.md` |

## Folder Structure

```
{PROJECT_OUTPUT}/
├── registry/
│   ├── pipeline-state.json          ← ONLY location for pipeline state
│   ├── fetched-stories.json         ← Story metadata registry
│   ├── fetched-epics.json           ← Epic metadata registry
│   └── bugs-log.json                ← Append-only log of reported bugs
├── stories/
│   ├── raw/{STORY-KEY}.raw.json     ← Raw API response per story
│   └── parsed/{STORY-KEY}.parsed.json ← Extracted AC + technical details
├── epics/
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
├── screenshots/{EPIC-KEY}/          ← Organized by EPIC, not story
├── bugs/drafts/{STORY-KEY}-draft.md ← Auto-saved on createJiraIssue failure
├── config/source-config.md          ← Copied from template at registration
└── ExtraResources/{KEY}/            ← Supplementary docs placed by user
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
| Bug Reporter | `test-cases/`, `registry/` | `registry/bugs-log.json`, `bugs/drafts/` |
| Orchestrator | `registry/` (all) | `registry/` (all) |

## Initialization Checklist (at project registration)

**Folders:** registry/, stories/raw/, stories/parsed/, epics/raw/, epics/parsed/, context/, strategy/strategy-versions/, test-cases/, tracking/archive/, tracking/reviews/, screenshots/, bugs/drafts/, config/, ExtraResources/

**Files:**
- `registry/pipeline-state.json` → initialized with schema from orchestrator
- `registry/fetched-stories.json` → `{ "schema_version": "1.0", "last_updated": "", "stories": [] }`
- `registry/fetched-epics.json` → `{ "schema_version": "1.0", "last_updated": "", "epics": [] }`

## Path Resolution Examples

**Correct:**
- Screenshot: `{PROJECT_OUTPUT}/screenshots/H20-37/H20-37-screenshot-001.png`
- TC file: `{PROJECT_OUTPUT}/test-cases/H20-52-test-cases.csv`
- Pipeline state: `{PROJECT_OUTPUT}/registry/pipeline-state.json`
- Parsed story: `{PROJECT_OUTPUT}/stories/parsed/H20-52.parsed.json`

**Incorrect (common mistakes):**
- ✗ Screenshot in story folder: `screenshots/H20-52/...` (should be epic H20-37)
- ✗ TC in subfolder: `test-cases/H20-52/H20-52-test-cases.csv` (should be flat)
- ✗ State in tracking: `tracking/registry/pipeline-state.json` (wrong location)
- ✗ Per-story test data: `test-cases/H20-52-test-data.md` (must be centralized)
