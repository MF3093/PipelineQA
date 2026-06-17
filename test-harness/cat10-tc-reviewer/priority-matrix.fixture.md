# Priority Matrix — QA Pipeline Tests (TC Reviewer Adversarial)
**Version:** 1
**Last Updated:** 2026-06-10
**Approved:** 2026-06-10

---

## Scope
- TEST-EPIC-TCR: Aircraft registration — form display, validation, submit, edit flows

## Out of Scope
- Delete flow (not yet in scope)

---

## Priority Matrix

| Rank | Story Key | Risk Description | Severity | Likelihood | Score | Dependency Weight | Functional Role | Open Items |
|------|-----------|-----------------|----------|------------|-------|-------------------|-----------------|------------|
| 1 | TEST-TCR-C | Submit failure — record not saved, edit flow becomes untestable | 3 | 1 | 3 | 0 | Interaction | — |
| 2 | TEST-TCR-B | Validation defect — invalid data accepted or valid data rejected | 3 | 1 | 3 | 0 | Required Content | — |
| 3 | TEST-TCR-A | Layout defect — fields missing or misarranged | 2 | 1 | 2 | 0 | Visual-Cosmetic | — |
| 4 | TEST-TCR-D | Edit defect — pre-fill or PUT call fails | 2 | 1 | 2 | 1 | Interaction | — |

---

## Batch History
| Batch | Date | Stories Added | Strategy Change |
|-------|------|---------------|-----------------|
| batch-001 | 2026-06-10 | TEST-TCR-A, TEST-TCR-B, TEST-TCR-C, TEST-TCR-D | Initial |
