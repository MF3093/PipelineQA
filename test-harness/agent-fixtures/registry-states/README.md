# Registry State Fixtures

Pre-configured `pipeline-state.json` and registry files for testing specific agent behaviors
without running a full pipeline.

## How to Use

1. Copy the desired fixture to `{PROJECT_OUTPUT}/registry/`.
2. Rename it to the expected filename (e.g. `pipeline-state.json`).
3. Load the relevant agent and observe behavior.
4. **Restore** the original file after the test.

> Keep a backup of your real `registry/` folder before using any of these fixtures.

## Fixture Index

| File | Simulates | Tests |
|------|-----------|-------|
| `pipeline-state-stale-lock.json` | `lock.locked = true` from a previous session | CRA-003, ORCH-N5 |
| `pipeline-state-no-context.json` | Clean state — context not yet approved | ORCH-N1, STRAT-N1, BYP-004 |
| `pipeline-state-context-approved-no-strategy.json` | Context approved, strategy NOT approved | ORCH-N2, TCG-N3 |
| `pipeline-state-strategy-stale-version.json` | Strategy approved BEFORE latest context update (`strategy_approved_at < context_approved_at`) | ORCH-N6 |
| `pipeline-state-crash-current-story.json` | Session crashed mid-TC-generation (`current_story` is set) | CRA-001, ORCH-N14 |
| `fetched-stories-mixed-status.json` | Batch with mixed statuses: 2 "parsed" + 1 "fetched" | GAP-C3, TCG-N1 |
