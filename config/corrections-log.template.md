# Corrections Log

> This file records every manual correction made at an approval gate during pipeline runs.
> It serves as evidence of human quality governance over AI-generated outputs.

---

## How to Use

At each approval gate, if you select **edit** or provide corrections, record the correction here using the format below. One entry per correction.

---

## Entry Format

```
### COR-{NNN}

- **Date:** {YYYY-MM-DD}
- **Run ID:** {run-YYYYMMDD-NNN}
- **Gate:** {Gate 2 | Gate 3 | Gate 4}
- **Story Key:** {STORY-KEY or N/A}
- **Category:** {see categories below}
- **What AI produced:** {brief description of the incorrect output}
- **What was corrected:** {brief description of the correction applied}
- **Root cause:** {why the AI got it wrong — missing context, ambiguous AC, hallucinated element, etc.}
- **Hardening applied:** {rule added, prompt adjusted, or N/A if no systemic fix}
```

---

## Correction Categories

| Category | Description |
|----------|-------------|
| `hallucinated-element` | AI invented a UI element, field name, or label not in any source |
| `missed-edge-case` | A valid test scenario was omitted |
| `wrong-priority` | Risk score or priority assignment was incorrect |
| `format-error` | Output did not match required CSV/import format |
| `duplicate-tc` | Generated TC was redundant with another |
| `wrong-expected-result` | Expected outcome in a TC step was incorrect |
| `scope-violation` | TC covered out-of-scope functionality |
| `missing-precondition` | TC lacked a required precondition or test data reference |
| `incorrect-merge` | ACs were merged when they should not have been (or vice versa) |
| `context-drift` | AI used outdated or wrong project context information |
| `other` | Does not fit above categories — describe in root cause |

---

## Log

<!-- Entries below this line -->

