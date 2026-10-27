# Test Harness — QA Pipeline

Self-contained fixture library for testing all agents in the QA multi-agent pipeline.
Every test can be reproduced independently — no live Jira connection required
(except tests explicitly marked **Live Jira**).

**This folder contains test definitions and fixtures only.** Test results are logged in
[`docs/adversarial-testing.md`](../docs/adversarial-testing.md). Step-by-step reproduction
instructions are in [`docs/testing-runbook.md`](../docs/testing-runbook.md).
See [`DEFERRED.md`](DEFERRED.md) for tests specified but not yet executed.

---

## Folder Structure

```
test-harness/
├── cat1-injection/         INJ-001 to INJ-008 — prompt injection via story/epic fields and external files
├── cat2-malformed/         MAL-001 to MAL-008 — malformed/edge-case story inputs
├── cat3-state-corruption/  CRA-001 to CRA-005 — filesystem state manipulation steps
├── cat4-gate-bypass/       BYP-001 to BYP-004 — approval gate bypass attempts
├── cat5-orchestrator/      ORC-001 to ORC-013 — project registration, run menu, & quality scenarios
├── cat6-story-analyzer/    SAA-001 to SAA-009 — story analysis adversarial tests
├── cat7-tc-generator/      TCG-001 to TCG-009 — TC generation adversarial tests
├── cat8-strategy/          STR-001 to STR-007 — story prioritizer adversarial tests
├── cat9-context-builder/   CTX-001 to CTX-006 — context builder adversarial tests
├── cat10-tc-reviewer/      TCR-001 to TCR-004 — TC reviewer adversarial tests
├── cat11-data-integrity/   DIN-001 to DIN-005 — data integrity & orphaned file detection (Cat 20)
├── cat12-fetcher/          FET-001 to FET-008 — fetcher adversarial tests
├── cat15-gate-injection/   GAT-001 to GAT-006 — gate prompt injection & XSS scenarios
├── cat16-security/         SEC-001 to SEC-003 — credential detection & PII handling
├── cat17-integration/      INT-001 to INT-003 — dependency validation & version consistency
├── cat18-story-edge-cases/ STO-001 to STO-004 — circular dependencies, special chars, edge cases
├── parser-formats/         FMT-001 to FMT-011 — story format variant fixtures (11 formats)
├── orc-001-all-features/   Full project setup with all optional features
├── orc-002-minimal/        Minimal project setup (no optional features)
└── agent-fixtures/
    ├── registry-states/    Pre-configured pipeline-state / registry JSON files
    └── parsed-stories/     Pre-built parsed story fixtures for TCG / TCR tests
```

---

## How to Run a Test

1. **Back up** your `{PROJECT_OUTPUT}/registry/` folder
2. **Copy** the fixture file(s) to the correct location (see category README for paths)
3. **Add registry entries** if required (e.g., `fetched-stories.json` with `status: "fetched"`)
4. **Invoke** the agent under test
5. **Observe** behavior against the expected behavior in the category README
6. **Record** results in [`docs/adversarial-testing.md`](../docs/adversarial-testing.md)
7. **Restore** your backup

---

## File Naming Conventions

| Pattern | Meaning |
|---------|---------|
| `{ID}.raw.json` | Raw Jira snapshot (Fetcher/Parser input) |
| `{ID}.parsed.json` | ParsedStory JSON (downstream agent input) |
| `{ID}-pipeline-state.json` | Pipeline state fixture |
| `{ID}-registry.fixture.json` | Registry (fetched-stories/epics) fixture |
| `{ID}-approved-matrix.md` | Pre-approved priority matrix |
| `{ID}-test-cases.csv` | Test case CSV fixture |
| `README.md` | Per-category test specifications |

---

## Test Count Summary

| Category | Folder | Agent(s) | Scenarios |
|----------|--------|----------|-----------|
| Cat 1 — Prompt Injection | `cat1-injection/` | Parser, Fetcher, Story Analyzer, TC Generator | 8 |
| Cat 2 — Malformed Input | `cat2-malformed/` | Parser, Fetcher | 8 |
| Cat 3 — Crash Recovery | `cat3-state-corruption/` | Orchestrator | 5 |
| Cat 4 — Gate Bypass | `cat4-gate-bypass/` | Orchestrator | 4 |
| Cat 5 — Orchestrator | `cat5-orchestrator/` | Orchestrator | 13 |
| Cat 6 — Story Analyzer | `cat6-story-analyzer/` | Story Analyzer | 9 |
| Cat 7 — TC Generator | `cat7-tc-generator/` | TC Generator | 9 |
| Cat 8 — Strategy | `cat8-strategy/` | Story Prioritizer | 7 |
| Cat 9 — Context Builder | `cat9-context-builder/` | Context Builder | 6 |
| Cat 10 — TC Reviewer | `cat10-tc-reviewer/` | TC Reviewer | 4 |
| Cat 12 — Fetcher | `cat12-fetcher/` | Fetcher | 8 |
| Cat 13 — Execution Efficiency | `cat13-execution/` | All (observation) | 6 |
| Cat 14 — Parser Formats | `parser-formats/` | Parser | 11 |
| Cat 15 — Gate Injection | `cat15-gate-injection/` | Orchestrator, Context Builder | 6 |
| Cat 16 — Security | `cat16-security/` | TC Generator, Parser | 3 |
| Cat 17 — Integration | `cat17-integration/` | Orchestrator, TC Generator | 3 |
| Cat 18 — Story Edge Cases | `cat18-story-edge-cases/` | Parser, Story Analyzer | 4 |
| Cat 19 — Project Creation | `orc-001-all-features/`, `orc-002-minimal/` | Orchestrator | 2 |
| Cat 20 — Data Integrity | `cat11-data-integrity/` | Orchestrator, Story Analyzer | 5 |
| **TOTAL** | | | **121** |
