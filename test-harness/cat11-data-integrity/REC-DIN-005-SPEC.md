# REC-DIN-005 — Assumption ID Uniqueness Validation

**Severity:** 🟡 MEDIUM  
**Category:** Data Integrity / Assumption Tracking  
**Test ID:** DIN-005  
**Date Identified:** 2026-06-22 v3.3

---

## Problem Statement

assumptions.md tracks open questions, assumptions, and blockers using ID prefixes:
- **A-NNN:** Assumptions (A-001, A-002, ...)
- **Q-NNN:** Questions (Q-001, Q-002, ...)
- **D-NNN:** Discrepancies (D-001, D-002, ...)
- **BLK-NNN:** Blockers (BLK-001, BLK-002, ...)

**Gap:** No automated duplicate ID detection. The `assumption-tracker` skill does NOT validate that a proposed ID is unique before logging new entries. Duplicate A-NNN or Q-NNN IDs can be created, causing:
- ID reference ambiguity (which A-001 is meant?)
- Broken links in agent outputs (references "See A-001" → multiple matches)
- Downstream parsing errors in tools that read assumptions.md

**Test Evidence:**
- DIN-005 test: Created two A-DIN-005-001 entries in assumptions.md with different content
- Both written successfully without duplicate ID alert
- No validation before logging occurred

---

## Root Causes

1. **No pre-write validation** — assumption-tracker does NOT read assumptions.md before logging new entries
2. **Manual markdown editing** — assumptions.md is a plain markdown file; no schema enforcement
3. **No uniqueness check** — assumed IDs are unique; no actual verification
4. **Multiple entry points** — any agent can call assumption-tracker; each call is independent

---

## Implementation Specification

### Location: assumption-tracker Skill

**Enhance assumption-tracker to validate ID uniqueness before logging:**

```
When logging new assumption/question/discrepancy/blocker:

1. Read assumptions.md from {PROJECT_OUTPUT}/tracking/assumptions.md
2. Extract all existing IDs using regex:
   - A-[0-9]{3} for assumptions
   - Q-[0-9]{3} for questions  
   - D-[0-9]{3} for discrepancies
   - BLK-[0-9]{3} for blockers
3. Build set of existing IDs: {A-001, A-002, A-003, Q-001, Q-002, ...}
4. Check if proposed ID exists in set
5. If duplicate found:
   Alert: "ID {ID} already exists in assumptions.md. Provide new ID or override? 
           (new-id / override / cancel)"
   - new-id   → User provides replacement ID (e.g., "A-004") → re-validate → log with new ID
   - override → Log anyway, flag as potential duplicate in file comment: "# WARNING: Duplicate ID {ID} exists"
   - cancel   → Do not log; return error to calling agent
6. If unique: log with confirmed ID
```

### Data Structure

assumptions.md entries must follow strict format to enable regex parsing:

```markdown
## Assumptions

### A-001
**Source:** {agent} Step {N}
**Status:** Open | Resolved
**Details:** {description}

### A-002
...
```

Regex patterns to extract IDs:
- Assumptions: `^### A-(\d{3})$`
- Questions: `^### Q-(\d{3})$`
- Discrepancies: `^### D-(\d{3})$`
- Blockers: `^### BLK-(\d{3})$`

---

## Testing & Verification

### Test Case: DIN-005 (Updated Spec)

**Setup:**
1. Create assumptions.md with existing entry: A-001
2. Call assumption-tracker to log new assumption with ID "A-001" (duplicate)

**Expected Behavior:**
- assumption-tracker reads assumptions.md
- Detects A-001 already exists
- Alerts user: "ID A-001 already exists. Provide new ID? (new-id / override / cancel)"
- On new-id: User provides "A-002" → re-validated → logged as A-002
- On override: Logged as A-001 with duplicate warning comment
- On cancel: Not logged; error returned to agent

**Pass Criteria:**
- Duplicate detection triggers before write
- User choice honored (new-id / override / cancel)
- No silent duplicate creation
- File integrity preserved

---

## Implementation Priority

**Phase 1 (Immediate):**
- assumption-tracker: read assumptions.md, extract existing IDs
- Implement duplicate check before all logging operations

**Phase 2 (Near-term):**
- Auto-increment: if user chooses new-id and doesn't specify, auto-suggest next available (A-003, A-004, ...)
- Batch logging: handle multiple assumptions logged in one call (check all for duplicates first)

**Phase 3 (Future):**
- assumptions.md schema validation on read (structured format enforcement)
- Circular assumption dependencies (A-001 references Q-002, Q-002 references A-001)

---

## Migration Path

### Current State (v1.0):
- Manual markdown editing
- No duplicate detection
- Caller responsible for ID uniqueness

### v2.0 (Immediate):
- Automated duplicate check before logging
- Alert user on collision
- Offer new-id / override / cancel

### v3.0 (Future):
- Structured assumptions.md format (YAML or JSON instead of markdown)
- Full schema validation on read/write
- ID auto-generation (assumption-tracker assigns next available ID)

---

## Security Impact

- **Before:** Duplicate IDs can cause agent references to ambiguously match multiple entries; malicious users could create false entries with existing IDs to override legitimate ones
- **After:** Duplicate prevention ensures ID uniqueness; references are unambiguous; data integrity maintained
- **Risk Level:** Reduces ID poisoning / collision from **MEDIUM** to **LOW**

