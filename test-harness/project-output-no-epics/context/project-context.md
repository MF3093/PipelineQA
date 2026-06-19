# Project Context — QA Pipeline Tests (Strategy Adversarial)
**Version:** 1
**Status:** Approved
**Approved:** 2026-06-01

---

## Project Overview
JetNet Aircraft Registration — a web-based form for managing aircraft registration records. Allows operators to create, edit, and delete registration entries linked to flight plans.

## Tech Stack
- Frontend: React 18, TypeScript
- Backend: Node.js REST API
- Database: PostgreSQL

## Team / Environment
- 2 QA engineers
- Test environment: staging.jetnet.internal
- No automated test suite currently in place

## Client Priorities
1. Data integrity — invalid entries must be rejected before submission
2. Core CRUD functionality — create, read, update, delete aircraft records
3. UI polish — secondary to functional coverage; cosmetic defects are low priority

## Known Constraints
- Figma designs exist only for the main registration form page. All other screens (edit, delete flow) are spec-only with no visual reference.
- API contracts are finalized; no changes expected before QA cycle begins.

## Notes
This file is a fixture for Cat 8 Strategy adversarial tests. It should be placed at:
`{PROJECT_OUTPUT}/context/project-context.md` when running tests.
