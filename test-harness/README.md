# Test Harness — QA Pipeline

Self-contained fixture library for testing all agents in the QA multi-agent pipeline.
Every test in this harness can be reproduced independently — no live Jira connection required
(except tests explicitly marked **Live Jira**).

---

## Folder Structure

```
test-harness/
├── cat1-injection/         INJ-001 to INJ-005 — prompt injection via story/epic fields
├── cat2-malformed/         MAL-001 to MAL-007 — malformed/edge-case story inputs
├── cat3-state-corruption/  CRA-001 to CRA-005 — filesystem state manipulation steps
├── cat4-gate-bypass/       BYP-001 to BYP-004 — approval gate bypass attempts
├── parser-formats/         FMT-001 to FMT-005 — story format variant fixtures
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
4. Compare actual behavior against the **Expected** column in
   `Documents/qa-pipeline-testing-plan.md`.
5. Record **Actual Result** in `docs/adversarial-testing.md` (Cat 1–4) or in the
   per-agent test results section of your corrections log.

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
| Cat 1 — Injection | `cat1-injection/` | 5 |
| Cat 2 — Malformed Input | `cat2-malformed/` | 7 |
| Cat 3 — State Corruption | `cat3-state-corruption/` | 5 |
| Cat 4 — Gate Bypass | `cat4-gate-bypass/` | 4 |
| Parser Format Coverage | `parser-formats/` | 5 |
| Agent Fixtures (per-unit tests) | `agent-fixtures/` | supports 119 scenarios |
