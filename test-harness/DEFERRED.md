# Deferred & Pending Tests

Tests that are specified but not yet executed, or rejected by design decision.

| Test ID | Category | Status | Reason | Prerequisite | Date |
|---------|----------|--------|--------|--------------|------|
| FET-006 | Cat 12 — Fetcher | Deferred | Out of scope for audit; requires MCP timeout simulation | MCP tool simulation or network throttling | 2026-06-22 |
| FET-007 | Cat 12 — Fetcher | Deferred | Out of scope for audit; requires 404 API response simulation | MCP tool simulation | 2026-06-22 |
| FET-008 | Cat 12 — Fetcher | Deferred | Out of scope for audit; requires 429 rate-limit simulation | MCP tool simulation | 2026-06-22 |
| SEC-003 | Cat 16 — Security | Pending | REC-SEC-003 hardening implemented; test not yet run | Rule 6 credential patterns active | 2026-06-22 |
| STO-002 | Cat 18 — Story Edge Cases | Rejected | Circular dependencies are a data quality issue, not a system failure; Story Analyzer catches contradictions | — | 2026-06-22 |
| STO-003 | Cat 18 — Story Edge Cases | Deferred | No execution needed; parser has no AC count limit | Fixture with 75 ACs | 2026-06-22 |
| FMT-005 | Cat 14 — Parser Formats | Deferred | Requires batch orchestration mode | Batch mode implementation | 2026-06-22 |
| REC-DIN-005 | Cat 20 — Data Integrity | Deferred (low priority) | Duplicate assumption ID detection; rarely occurs in practice | assumption-tracker skill update | 2026-06-22 |

Specifications for deferred tests are in their respective category folders (e.g., `cat12-fetcher/FET-CRITICAL-TESTS.md`).
