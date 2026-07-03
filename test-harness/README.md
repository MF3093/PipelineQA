# Test Harness — QA Pipeline

Self-contained fixture library for testing all agents in the QA multi-agent pipeline.
Every test in this harness can be reproduced independently — no live Jira connection required
(except tests explicitly marked **Live Jira**).

---

## Folder Structure

```
test-harness/
├── cat1-injection/         INJ-001 to INJ-007 — prompt injection via story/epic fields and external files
├── cat2-malformed/         MAL-001 to MAL-008 — malformed/edge-case story inputs
├── cat3-state-corruption/  CRA-001 to CRA-005 — filesystem state manipulation steps
├── cat4-gate-bypass/       BYP-001 to BYP-004 — approval gate bypass attempts
├── cat5-orchestrator/      ORC-001 to ORC-004 — project registration & quality scenarios
├── cat6-story-analyzer/    SAA-001 to SAA-008 — story analysis adversarial tests
├── cat7-tc-generator/      TCG-001 to TCG-009 — TC generation adversarial tests
├── cat8-strategy/          STR-001 to STR-007 — story prioritizer adversarial tests
├── cat9-context-builder/   CTX-001 to CTX-006 — context builder adversarial tests
├── cat10-tc-reviewer/      TCR-001 to TCR-004 — TC reviewer adversarial tests
├── cat12-fetcher/          FET-001 to FET-005 — fetcher adversarial tests
├── cat11-data-integrity/   DIN-001 to DIN-005 — data integrity & orphaned file detection
├── cat15-gate-injection/   GAT-001 to GAT-006 — gate prompt injection & XSS scenarios
├── cat16-security/         SEC-001 to SEC-003 — credential detection & PII handling
├── cat17-integration/      INT-001 to INT-003 — dependency validation & version consistency
├── cat18-story-edge-cases/ STO-001 to STO-004 — circular dependencies, special chars, edge cases
├── parser-formats/         FMT-001 to FMT-011 — story format variant fixtures (11 formats)
├── orc-001-all-features/   Full project setup with all optional features (epics, screenshots, resources)
├── orc-002-minimal/        Minimal project setup (no epics, no screenshots, no extra resources)
└── agent-fixtures/
    ├── registry-states/    Pre-configured pipeline-state / registry JSON files
    └── parsed-stories/     Pre-built parsed story fixtures for TCG / TCR tests
```

---

## How to Use a Fixture

1. Copy the fixture `.raw.json` to the project output folder:
   - Story raws → `{PROJECT_OUTPUT}/stories/raw/`
   - Epic raws  → `{PROJECT_OUTPUT}/epics/raw/`
2. If a registry fixture is included, copy it to `{PROJECT_OUTPUT}/registry/` and rename
   to the expected filename (e.g. `pipeline-state.json`).
3. Load the relevant agent in Copilot Chat and run the scenario.
4. Compare actual behavior against expected results documented in evidence-summary.md or the
   per-category README in each test folder.
5. Record results in your project's corrections log.

---

## Naming Conventions

| Prefix | Type |
|--------|------|
| `INJ-NNN` | Injection payload fixture |
| `MAL-NNN` | Malformed story/epic fixture |
| `FMT-NNN` | Parser format variant fixture |
| `registry-*.json` | Pipeline state / registry fixture |

---

## Test Count Summary

| Category | Folder | Scenarios |
|----------|--------|-----------|
| Cat 1 — Prompt Injection | `cat1-injection/` | 7 |
| Cat 2 — Malformed Input | `cat2-malformed/` | 8 |
| Cat 3 — Crash Recovery | `cat3-state-corruption/` | 5 |
| Cat 4 — Gate Bypass | `cat4-gate-bypass/` | 4 |
| Cat 5 — Output Quality | `cat5-orchestrator/` | 4 |
| Cat 6 — Story Analyzer Quality | `cat6-story-analyzer/` | 8 |
| Cat 7 — TC Generator Quality | `cat7-tc-generator/` | 9 |
| Cat 8 — Story Prioritizer Quality | `cat8-strategy/` | 7 |
| Cat 9 — Context Builder Quality | `cat9-context-builder/` | 6 |
| Cat 10 — TC Reviewer Quality | `cat10-tc-reviewer/` | 4 |
| Cat 12 — Fetcher Quality | `cat12-fetcher/` | 5 |
| Cat 14 — Parser Format Coverage | `parser-formats/` | 11 |
| Cat 15 — Gate & Interview Injection | (in agent tests) | 6 |
| Cat 16 — Security: Sensitive Data | (in agent tests) | 3 |
| Cat 17 — Integration & Consistency | (in agent tests) | 3 |
| Cat 18 — Story Content Edge Cases | (in agent tests) | 4 |
| Cat 19 — Orchestrator Project Creation | `orc-001-all-features/`, `orc-002-minimal/` | 2 |
| **TOTAL** | **All categories** | **119 tests** |
