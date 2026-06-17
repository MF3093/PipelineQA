# Priority Matrix — QA Pipeline Tests (Strategy Adversarial)
**Version:** 1
**Last Updated:** 2026-06-01
**Approved:** 2026-06-01

---

## Scope
What is being tested in this strategy:
- TEST-EPIC-STR: Aircraft registration form — core API integration (batch-001)

## Out of Scope
Items explicitly excluded from testing:
- Caching and offline behavior — explicitly excluded per story-level out_of_scope
- Adding new aircraft types through the registration form

---

## Priority Matrix

Scoring: (Severity × Likelihood) + Dependency Weight = Score
- Severity: 1=Low impact → 2=Medium impact → 3=High impact
- Likelihood: 1=Low probability → 2=Medium probability → 3=High probability
- Dependency Weight: 0=Standalone → 3=Critical blocker (how many other stories lose testability if this story fails — evaluated at strategy time, before TC generation)
- **Higher score = higher processing priority.** Ranked highest score first.

| Rank | Story Key | Risk Description | Severity | Likelihood | Score | Dependency Weight | Functional Role | Open Items |
|------|-----------|-----------------|----------|------------|-------|-------------------|-----------------|------------|
| 1 | TEST-STR-002A | Aircraft type options fail to load from API — Aircraft Type field is unpopulable, blocking all registration flows that require aircraft type selection | 3 | 1 | 3 | 0 | Required Content | — |

**Flagged stories (elevated risk):**
- None

---

## Batch History
| Batch | Date | Stories Added | Strategy Change |
|-------|------|---------------|-----------------|
| batch-001 | 2026-06-01 | TEST-STR-002A | Initial strategy created |

<!-- FIXTURE NOTE: This file is the pre-approved v1 matrix for STR-002 extension mode test.
     TEST-STR-002A has Dependency Weight = 0 (correct at batch-001 time — no dependents existed yet).
     After batch-002 adds TEST-STR-002B (which depends on TEST-STR-002A), the agent must:
       - Update ONLY Dependency Weight on the TEST-STR-002A row (0 → 1)
       - NOT modify Severity (3), Likelihood (1), Risk Description, or Functional Role
       - Produce a delta view showing the DW change before the approval gate
-->
