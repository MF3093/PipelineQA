# Category 13 — Live Pipeline Execution Efficiency

Behavioral observations made during real pipeline runs. These tests monitor agent tool-call
patterns for efficiency — they are not synthetic fixtures.

---

## What This Category Tests

Each test watches a specific agent's tool-call sequence during a live run and verifies it
matches the documented procedure. Failures are wasted calls (efficiency), not incorrect outputs.

---

## Test Scenarios

| Test ID | Agent | What to Watch |
|---------|-------|---------------|
| RUN-001 | Orchestrator | `pipeline-state.json` and `fetched-stories.json` read at most once per phase (unless a write occurred between reads) |
| RUN-002 | Orchestrator | At startup Step 2: `Test-Path` is the first check — no `file_search` or `Get-ChildItem -Recurse` |
| RUN-003 | Fetcher | `tool_search` is the very first action before any Rovo Search or semantic_search |
| RUN-004 | Parser | Parser does NOT read raw/parsed files outside the current batch |
| RUN-005 | Parser | `fetched-stories.json` read via `read_file` only — no `grep_search` or `file_search` |
| RUN-006 | TC Generator | `fetched-stories.json` and `pipeline-state.json` read via `read_file` only |

---

## How to Run

1. Copy `RUN-baseline-pipeline-state.fixture.json` to `{PROJECT_OUTPUT}/registry/pipeline-state.json`
2. Ensure `stories/parsed/` has at least 3 parsed stories
3. Start a full or partial pipeline run
4. Monitor the agent's tool calls in the session transcript
5. Record results in `docs/adversarial-testing.md` → Category 13

---

## Pass / Fail Criteria

**PASS:** Agent uses only the documented tool at the documented step.
**FAIL:** Extra tool call made before the required one, or registry file accessed via search instead of `read_file`.

Results logged in `docs/adversarial-testing.md` → Category 13.
