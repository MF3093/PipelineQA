# Priority Matrix — QA Pipeline Tests (Strategy Adversarial)
**Version:** 1
**Last Updated:** 2026-06-01
**Approved:** 2026-06-01

---

## Scope
What is being tested in this strategy:
- TEST-EPIC-STR: Aircraft registration form — core data entry, validation, and persistence (batch-001)

## Out of Scope
Items explicitly excluded from testing:
- Audit trail and change history — excluded per story-level out_of_scope
- Draft saving before submission

---

## Priority Matrix

Scoring: (Severity × Likelihood) + Dependency Weight = Score
- Severity: 1=Low impact → 2=Medium impact → 3=High impact
- Likelihood: 1=Low probability → 2=Medium probability → 3=High probability
- Dependency Weight: 0=Standalone → 3=Critical blocker (how many other stories lose testability if this story fails — evaluated at strategy time, before TC generation)
- **Higher score = higher processing priority.** Ranked highest score first.

| Rank | Story Key | Risk Description | Severity | Likelihood | Score | Dependency Weight | Functional Role | Open Items |
|------|-----------|-----------------|----------|------------|-------|-------------------|-----------------|------------|
| 1 | TEST-STR-003C | Submit failure — POST to /api/registrations fails, no record persisted; edit and delete flows both become untestable | 3 | 1 | 3 | 0 | Interaction | — |
| 2 | TEST-STR-003B | Validation logic defect — invalid registration numbers accepted or valid ones incorrectly rejected, allowing dirty data into the system | 3 | 1 | 3 | 0 | Required Content | — |
| 3 | TEST-STR-003A | Display defect — missing title or incorrect default date causes user confusion but does not block data entry | 1 | 1 | 1 | 0 | Visual-Cosmetic | — |

**Flagged stories (elevated risk):**
- None

---

## Batch History
| Batch | Date | Stories Added | Strategy Change |
|-------|------|---------------|-----------------|
| batch-001 | 2026-06-01 | TEST-STR-003A, TEST-STR-003B, TEST-STR-003C | Initial strategy created |

<!-- FIXTURE NOTE: This file is the pre-approved v1 matrix for STR-003 extension mode test.
     All three rows have Dependency Weight = 0 (correct at batch-001 time).
     After batch-002 adds TEST-STR-003D and TEST-STR-003E (both depend on TEST-STR-003C),
     the agent must:
       - Update Dependency Weight on TEST-STR-003C row (0 → 2)
       - Update Score on TEST-STR-003C row ((3×1)+2 = 5)
       - NOT modify Severity, Likelihood, or Risk Description on any existing row
       - Write scoped_story_keys with ALL FIVE keys to pipeline-state.json on approval:
         ["TEST-STR-003A", "TEST-STR-003B", "TEST-STR-003C", "TEST-STR-003D", "TEST-STR-003E"]
     FAILURE MODE: agent writes only ["TEST-STR-003D", "TEST-STR-003E"] to scoped_story_keys
     (forgetting the batch-001 keys), breaking downstream routing for stories A, B, C.
-->
