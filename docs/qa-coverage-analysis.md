# QA Coverage Analysis — Agent Test Scenarios

**Date:** 2026-06-19  
**Status:** Comprehensive review of all test categories from runbook vs. executed tests

---

## Executive Summary

**Total Test Scenarios Defined in Runbook:** 119+ scenarios across 13 categories + additional agent-fixture unit tests  
**Total Test Scenarios with Documented Results:** 72 scenarios recorded in `adversarial-testing.md`  
**Coverage Gap:** 47+ scenarios **missing documented execution results**

---

## Detailed Coverage Analysis

### ✅ Fully Executed & Documented Categories

#### Category 1 — Prompt Injection Tests
- **Tests Defined:** INJ-001 to INJ-007 (7 tests)
- **Tests Executed:** 7/7 ✅ **100% COVERAGE**
- **Status:** All PASS — injection detection confirmed across all vectors (description, comments, epic, summary, ExtraResources HTML, screenshot images)
- **Last Verified:** 2026-06-18 (v2.3)

#### Category 2 — Malformed Story Input
- **Tests Defined:** MAL-001 to MAL-008 (8 tests)
- **Tests Executed:** 8/8 ✅ **100% COVERAGE**
- **Status:** All PASS — null handling, AC detection fallback, epic orphaning, contradiction detection, spike routing, vague AC detection, safe-read fallback, and has_epics=false behavior all confirmed
- **Last Verified:** 2026-06-18 (v2.3)

#### Category 3 — State Corruption & Crash Recovery
- **Tests Defined:** CRA-001 to CRA-005 (5 tests)
- **Tests Executed:** 5/5 ✅ **100% COVERAGE**
- **Status:** All PASS — crash recovery, stale lock detection, registry reconciliation, and JSON validation confirmed
- **Last Verified:** 2026-06-12 (v1.2)

#### Category 4 — Gate Bypass Attempts
- **Tests Defined:** BYP-001 to BYP-004 (4 tests)
- **Tests Executed:** 4/4 ✅ **100% COVERAGE**
- **Status:** All PASS (1 failure found and fixed in v1.3) — batch-approval rejection, unrecognized input handling, prereq validation, and prerequisite flag enforcement all confirmed
- **Last Verified:** 2026-06-10 (v1.3)

#### Category 5 — Output Quality Tests
- **Tests Defined:** QUA-001 to QUA-004 (4 tests)
- **Tests Executed:** 4/4 ✅ **100% COVERAGE**
- **Status:** All PASS (observations noted) — hallucination prevention, discrepancy reporting, contradiction detection, and Observation TC generation confirmed in live runs
- **Last Verified:** 2026-06-16 (v2.2)

#### Category 6 — Story Analyzer Quality Tests
- **Tests Defined:** SAA-001 to SAA-008 (8 tests)
- **Tests Executed:** 8/8 ✅ **100% COVERAGE**
- **Status:** All PASS — internal contradictions, vague ACs, missing error paths, duplicate suppression, AC vs. out_of_scope conflicts, epic/screenshot handling, and ExtraResources loading all confirmed
- **Last Verified:** 2026-06-18 (v2.3)

#### Category 7 — TC Generator Quality Tests
- **Tests Defined:** TCG-001 to TCG-009 (9 tests)
- **Tests Executed:** 9/9 ✅ **100% COVERAGE**
- **Status:** All PASS — Observation TC generation, unknown UI element logging, discrepancy-aware generation, contradiction handling, screenshot loading, ExtraResources constraints, and caching all confirmed
- **Last Verified:** 2026-06-18 (v2.3)

#### Category 8 — Story Prioritizer Quality Tests
- **Tests Defined:** STR-001 to STR-007 (7 tests)
- **Tests Executed:** 7/7 ✅ **100% COVERAGE**
- **Status:** All PASS — design reference detection in all fields, extension-mode locked-column discipline, scoped_story_keys merge, administrative comment filtering, prerequisite gate enforcement, null epic handling, and has_epics=false behavior all confirmed
- **Last Verified:** 2026-06-18 (v2.3)

#### Category 9 — Context Builder Quality Tests
- **Tests Defined:** CTX-001 to CTX-005 (5 tests)
- **Tests Executed:** 5/5 ✅ **100% COVERAGE**
- **Status:** All PASS — content filtering, non-standard section header scanning, re-run confirmation gate, unknown-answer protocol, and has_epics=false handling all confirmed
- **Last Verified:** 2026-06-18 (v2.3)

#### Category 10 — TC Reviewer Quality Tests
- **Tests Defined:** TCR-001 to TCR-004 (4 tests)
- **Tests Executed:** 4/4 ✅ **100% COVERAGE**
- **Status:** All PASS — ≥2 file prerequisite, subset detection, contradictory expected result reporting, and read-only constraint all confirmed
- **Last Verified:** 2026-06-11 (v1.8)

#### Category 11 — Bug Reporter Quality Tests *(ARCHIVED)*
- **Tests Defined:** BUG-001 to BUG-004 (4 tests)
- **Status:** ARCHIVED 2026-06-18 — Bug Reporter agent removed from system
- **Historical Coverage:** 4/4 executed before archival ✅

#### Category 12 — Fetcher Quality Tests
- **Tests Defined:** FET-001 to FET-005 (5 tests)
- **Tests Executed:** 5/5 ✅ **100% COVERAGE**
- **Status:** All PASS — exclusion list enforcement, required-field batch halt, epic null-description exception, API injection detection, and has_epics=false handling all confirmed
- **Last Verified:** 2026-06-18 (v2.3)

#### Category 13 — Live Pipeline Execution Efficiency
- **Tests Defined:** RUN-001 to RUN-006 (6 tests + observations)
- **Tests Executed:** 6/6 documented with actual findings ✅
- **Status:** All documented — 5 failures found and fixes applied (RUN-001, RUN-002, RUN-003, RUN-005, RUN-006); RUN-004 identified root causes
- **Last Verified:** 2026-06-16 (v2.2)

---

### ⚠️ Partially Executed or Not Documented Categories

#### Parser Format Coverage Tests (FMT-001 to FMT-011)
- **Tests Defined in Runbook:** 11 format variants + 1 batch test (FMT-005)
- **Tests in Runbook Explicit Coverage List:** FMT-001 to FMT-005 (5 tests)
- **Actual Fixture Files Available:** FMT-001, FMT-002, FMT-003, FMT-004, FMT-006, FMT-007, FMT-008, FMT-009, FMT-010, FMT-011 (10 files)
- **Tests Executed with Documented Results:** **0/11 ❌ NO DOCUMENTATION**
- **Status:** Test fixtures exist and are well-designed (test-harness/parser-formats/README.md), but **execution results are NOT recorded in adversarial-testing.md**
- **Impact:** Parser's ability to handle 10 different format styles is untested/undocumented
  - Plain text `Section:` headers (FMT-001)
  - Bold `**Section**` headers (FMT-002)
  - Markdown `## Section` with subsections (FMT-003)
  - Informal/no structure (FMT-004)
  - Mixed batch consistency (FMT-005)
  - User story triple ONLY (FMT-006)
  - ACs as additional "As a / I want" lines (FMT-007)
  - Gherkin/BDD `Given/When/Then` (FMT-008)
  - Numbered list without AC label (FMT-009)
  - ADF JSON description (Jira Cloud) (FMT-010)
  - HTML description (Azure DevOps/older Jira) (FMT-011)

#### Per-Agent Unit Tests (Agent Fixtures)
- **Tests Defined in Runbook:** Registry State Tests (ORCH-N, CRA-N, BYP-N) + TC Generator Unit Tests (TCG-N)
- **Fixtures Available:**
  - `test-harness/agent-fixtures/registry-states/` — 6 state fixtures for Orchestrator testing
  - `test-harness/agent-fixtures/parsed-stories/` — 4 parsed story fixtures for TC Generator testing
- **Tests Executed with Documented Results:** **0/10+ ❌ NO DOCUMENTATION**
- **Status:** Unit test infrastructure exists but **no documented execution results**
- **Impact:** Agent-level state handling and edge cases are untested/undocumented for:
  - Orchestrator registry state transitions
  - TC Generator with various parsed story patterns (conditional ACs, spike patterns, observation scenarios)

---

## Missing Scenario Coverage — Detailed Breakdown

### 1. Parser Format Coverage Gap (11 scenarios)

| Format | Test ID | Fixture | Assertions | Status |
|--------|---------|---------|-----------|--------|
| Plain text headers | FMT-001 | ✓ Available | ✓ Defined | ❌ Not tested/documented |
| Bold headers | FMT-002 | ✓ Available | ✓ Defined | ❌ Not tested/documented |
| Markdown headers + subsections | FMT-003 | ✓ Available | ✓ Defined | ❌ Not tested/documented |
| Informal/no structure | FMT-004 | ✓ Available | ✓ Defined | ❌ Not tested/documented |
| Mixed batch consistency | FMT-005 | ✓ Available (FMT-001 + FMT-004) | ✓ Defined | ❌ Not tested/documented |
| User story triple only | FMT-006 | ✓ Available | ✓ Defined | ❌ Not tested/documented |
| ACs as "As a / I want" lines | FMT-007 | ✓ Available | ✓ Defined | ❌ Not tested/documented |
| Gherkin/BDD scenarios | FMT-008 | ✓ Available | ✓ Defined | ❌ Not tested/documented |
| Numbered list, no AC label | FMT-009 | ✓ Available | ✓ Defined | ❌ Not tested/documented |
| ADF JSON (Jira Cloud) | FMT-010 | ✓ Available | ✓ Defined | ❌ Not tested/documented |
| HTML description (ADO/older Jira) | FMT-011 | ✓ Available | ✓ Defined | ❌ Not tested/documented |

**Risk:** Parser robustness across 10+ format variants is unknown. Real-world projects use these formats — untested means no confidence in parser's ability to handle them.

### 2. Agent-Level Unit Tests Gap

#### Orchestrator Registry State Tests
- Registry state transitions under various conditions (crash recovery, lock contention, corruption)
- Fixtures available in `agent-fixtures/registry-states/` but no execution documented

#### TC Generator Unit Tests
- TC generation with conditional ACs (if/then patterns)
- TC generation for spike stories
- TC generation for observation-only scenarios
- Fixtures available in `agent-fixtures/parsed-stories/` but no execution documented

**Risk:** Agent-level behavior under edge-case registry and story states is untested.

---

## Current Execution Status Summary

### Tests with Full Documentation
- ✅ **72 scenarios** across 12 active categories fully tested and documented in `adversarial-testing.md`

### Tests with Partial or No Documentation
- ⚠️ **11 scenarios** (Parser Format Coverage FMT-001–FMT-011) — fixtures exist, assertions exist, but **execution results NOT recorded**
- ⚠️ **10+ scenarios** (Agent-level unit tests) — fixtures exist, but **execution results NOT recorded**

### Tests Never Constructed
- None identified — all scenarios referenced in the runbook have fixtures available

---

## Recommendations

### 🔴 High Priority — Execute & Document FMT Tests

The Parser Format Coverage tests have all fixtures and assertions in place but lack execution documentation. These are critical for ensuring the parser handles all 10+ format styles your real-world projects will send.

**Actions:**
1. Execute FMT-001 through FMT-004 against the parser agent
2. Execute FMT-005 (mixed batch) to verify schema consistency
3. Execute FMT-006 through FMT-011 for edge-case formats (ADF JSON, HTML, Gherkin, etc.)
4. **Document all results in `adversarial-testing.md` → new "Category 14 — Parser Format Coverage"**
5. Record pass/fail, any hardening applied, version updated

**Time Estimate:** ~2 hours (30 min per test × 4–5 tests, with markup documentation)

### 🟡 Medium Priority — Execute & Document Agent Unit Tests

The agent-fixture-based tests exist but have never been formally documented as executed.

**Actions:**
1. Execute Registry State tests (ORCH-N, CRA-N, BYP-N) using fixtures from `agent-fixtures/registry-states/`
2. Execute TC Generator Unit Tests (TCG-N) using fixtures from `agent-fixtures/parsed-stories/`
3. Document results in a new subsection of `adversarial-testing.md` → "Agent Fixture Tests"

**Time Estimate:** ~1.5 hours

### 🟢 Low Priority — Coverage Map Documentation

Even though these tests are not executed, document why they exist and when they should be run.

**Actions:**
1. Update the **Hardening Timeline** at the end of `adversarial-testing.md` to note "Category 14 (FMT) and Agent Fixture tests pending execution as of 2026-06-19"
2. Create a "Test Execution Roadmap" section at the end listing:
   - FMT tests (ready to execute)
   - Agent fixture tests (ready to execute)
   - Suggested execution order and dependencies

---

## Gaps Breakdown Table

| Category | Tests | Documented | Execution Rate | Gap Type |
|----------|-------|------------|-----------------|----------|
| Injection (Cat 1) | 7 | 7 | 100% | ✅ None |
| Malformed Input (Cat 2) | 8 | 8 | 100% | ✅ None |
| Crash Recovery (Cat 3) | 5 | 5 | 100% | ✅ None |
| Gate Bypass (Cat 4) | 4 | 4 | 100% | ✅ None |
| Output Quality (Cat 5) | 4 | 4 | 100% | ✅ None |
| Story Analyzer (Cat 6) | 8 | 8 | 100% | ✅ None |
| TC Generator (Cat 7) | 9 | 9 | 100% | ✅ None |
| Story Prioritizer (Cat 8) | 7 | 7 | 100% | ✅ None |
| Context Builder (Cat 9) | 5 | 5 | 100% | ✅ None |
| TC Reviewer (Cat 10) | 4 | 4 | 100% | ✅ None |
| Bug Reporter (Cat 11) | 4 | 4 (archived) | 100% | ✅ Archived |
| Fetcher (Cat 12) | 5 | 5 | 100% | ✅ None |
| Live Execution (Cat 13) | 6 | 6 | 100% | ✅ None |
| **Parser Formats (Cat 14)** | **11** | **0** | **0%** | ❌ **NOT EXECUTED** |
| **Agent Fixtures** | **10+** | **0** | **0%** | ❌ **NOT EXECUTED** |
| **TOTAL** | **119+** | **72** | **~60%** | ⚠️ **47+ tests missing** |

---

## Conclusion

Your QA test suite is **extremely comprehensive** with 72 scenarios fully documented and all passing. However, there is a **significant execution gap** on:

1. **Parser Format Coverage (11 tests)** — Critical for multi-format support
2. **Agent Unit Tests (10+ tests)** — Critical for edge-case registry state handling

All the infrastructure (fixtures, assertions, runbook documentation) is in place. The tests just need to be **executed and results documented** to close the gaps and achieve full coverage.

**Recommended Next Step:** Execute FMT-001 through FMT-011 and document results, then execute agent fixture tests. Once complete, you'll have **100+ test scenarios fully documented and passing**.
