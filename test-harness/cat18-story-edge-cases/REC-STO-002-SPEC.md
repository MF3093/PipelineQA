# REC-STO-002 — Circular Dependency Detection

**Severity:** 🟡 HIGH  
**Category:** Integration / Story Dependencies  
**Test ID:** STO-002  
**Date Identified:** 2026-06-22 v2.8

---

## Problem Statement

Story-level dependency validation (REC-INT-002, Story Analyzer Step 3b) checks that each dependency is parsed before proceeding with analysis.

**Gap:** System does NOT detect transitive cycles (circular dependencies):
- Story A depends on Story B
- Story B depends on Story A
- System accepts both as "valid dependencies" — both are parsed

Result: Impossible execution order — TC generation cannot proceed for either story without the other being complete first.

**Test Evidence:**
- STO-002 test: Created Story-A→B→A cycle
- Story Analyzer Step 3b detected STO-002-B is parsed (✓)
- But did NOT detect STO-002-B also depends on STO-002-A (cycle)
- System would attempt TC generation on both stories despite impossible dependency chain

---

## Root Causes

1. **Per-story validation only** — REC-INT-002 checks dependencies are parsed, not transitive relationships
2. **No graph analysis** — system lacks cycle detection algorithm (DFS or similar)
3. **Batch-level visibility required** — detecting cycles requires full batch context, which Story Analyzer doesn't have
4. **No ordering enforcement** — even if cycle detected, no mechanism to reorder stories or halt strategically

---

## Implementation Specification

### Location: `orchestrator.agent.md` — New Step (Before TC Generation)

**Add as Step 14, after Step 13 (Re-fetch Protection), before Step 16 (TC Generation):**

```markdown
14. **CIRCULAR DEPENDENCY DETECTION (REC-STO-002):**
    **Applies to all runs with Phase 2 (TC Generation). Skip if Phase 1 only.**
    
    Build dependency graph for all stories in current batch:
    
    Algorithm (Depth-First Search for cycles):
    ```
    for each STORY-KEY in batch:
      visited = {}
      recursion_stack = {}
      
      function has_cycle(key):
        if key in recursion_stack:
          return true  // cycle detected
        if key in visited:
          return false // already checked, no cycle this path
        
        visited[key] = true
        recursion_stack[key] = true
        
        for each dependency in dependencies[key]:
          if has_cycle(dependency):
            return true
        
        recursion_stack[key] = false
        return false
    
    if has_cycle(STORY-KEY):
      cycle_found = true
      trace_cycle(STORY-KEY)  // find the cycle path
    ```
    
    On cycle detection:
    ```
    "Circular dependency detected: {CYCLE-PATH}"
    
    Example output:
    "Circular dependency detected: TEST-001 → TEST-002 → TEST-003 → TEST-001"
    
    Options:
      (break-cycle)   — manual: remove one dependency, restart batch
      (split-batch)   — remove cyclic stories from this batch, create new batch
      (halt)          — stop run, manual intervention required
    
    break-cycle → Ask: "Which story should be removed from dependencies to break the cycle?"
                  User provides {KEY to edit}, halt, re-run Parser with corrected dependencies.
    
    split-batch → Remove {CYCLE-STORY-KEYS} from current batch. 
                  Report: "{N} stories in cycle removed. Continue with {remaining} stories? (yes / no)"
                  yes → proceed with TC generation for non-cyclic subset
                  no  → stop run
    
    halt        → Release lock. STOP. Report cycle path. Require manual resolution.
    ```

### Data Structure

Store cycle detection results in working memory (not persisted):
```json
{
  "cycle_detected": true,
  "cycle_path": ["TEST-001", "TEST-002", "TEST-003", "TEST-001"],
  "involved_keys": ["TEST-001", "TEST-002", "TEST-003"],
  "user_action": "pending"
}
```

### TC Generation Skip Logic

**In TC Generation phase (after Step 14), add check:**

```markdown
If cycle_detected == true and user_action == "split-batch":
  Filter story list to exclude involved_keys
  TC Generation proceeds for remaining stories only
  
If cycle_detected == true and user_action == "halt":
  Do not proceed. Release lock. STOP.
```

---

## Detection Examples

### Example 1: Simple Cycle (A ↔ B)

```
Story-A:
  dependencies: ["Story-B"]

Story-B:
  dependencies: ["Story-A"]

Detection: A → B → A (cycle)
Path length: 2
Severity: CRITICAL
```

### Example 2: Transitive Cycle (A → B → C → A)

```
Story-A:
  dependencies: ["Story-B"]

Story-B:
  dependencies: ["Story-C"]

Story-C:
  dependencies: ["Story-A"]

Detection: A → B → C → A (cycle)
Path length: 3
Severity: CRITICAL
```

### Example 3: No Cycle (Valid DAG)

```
Story-A:
  dependencies: ["Story-B", "Story-C"]

Story-B:
  dependencies: ["Story-C"]

Story-C:
  dependencies: []

Graph: A → B → C (no cycle)
Detection: None
Proceed: TC generation allowed for all
```

---

## Testing & Verification

### Test Case: STO-002 (Updated Spec)

**Setup:**
1. Story-A with dependencies: ["Story-B"]
2. Story-B with dependencies: ["Story-A"]
3. Run Orchestrator with both stories in batch

**Expected Behavior:**
- Step 14 cycle detection triggers
- Cycle path identified: A → B → A
- Alert shown: "Circular dependency detected: TEST-A → TEST-B → TEST-A"
- User options presented
- If "split-batch": TEST-A and TEST-B removed from TC generation
- If "halt": pipeline stops

**Pass Criteria:**
- Cycle correctly identified
- Path correctly traced
- User options provided
- TC generation prevented for cyclic stories
- Processing continues for non-cyclic stories (if split-batch chosen)

---

## Implementation Complexity

| Aspect | Complexity | Notes |
|--------|-----------|-------|
| **Graph construction** | LOW | Walk dependencies[] from parsed stories |
| **Cycle detection** | MEDIUM | DFS algorithm, O(V+E) complexity |
| **Path tracing** | MEDIUM | Backtrack recursion stack to show cycle |
| **User options** | MEDIUM | Handle 3 paths (break-cycle, split-batch, halt) |
| **Testing** | HIGH | Edge cases: self-loops, partial cycles, transitive |

---

## Edge Cases

### Self-Loop (Story depends on itself)

```
Story-A:
  dependencies: ["Story-A"]

Detection: A → A (self-loop)
Handling: Same as cycle detection
Alert: "Story A has a self-reference in dependencies"
```

### Partial Cycle in Larger Batch

```
Batch: [A, B, C, D, E]

Dependencies:
  A → B
  B → C
  C → B (cycle: B ↔ C)
  D → A
  E → (no deps)

Detection: Cycle found in subset [B, C]
Handling: split-batch removes [B, C], continues with [A, D, E]
```

### Self-Referential Comment (Not a Cycle)

```
Story-A:
  dependencies: []
  comments: ["See Story-A for more details"]

Detection: None (comments are not dependencies)
Handling: Proceed normally
```

---

## Performance Notes

- **Graph construction:** O(N) where N = stories in batch (parse dependencies[])
- **Cycle detection:** O(V+E) DFS where V = stories, E = dependency edges
- **Typical batch:** 20 stories, ~30-50 edges → ~100ms cycle detection
- **Worst case:** 100 stories, all interconnected → ~1sec (acceptable)

---

## Migration Path

### Phase 1 (Immediate): Detection Only
- Implement cycle detection
- Alert user on cycle found
- Require manual intervention (no auto-split)

### Phase 2 (Near-term): Auto-Split
- Implement split-batch option
- Automatically remove cyclic subset
- Continue with valid stories

### Phase 3 (Future): Cycle Breaking Suggestions
- Suggest which dependency to remove to break cycle
- Offer to re-parse story with corrected dependencies
- Auto-retry batch

---

## Security Impact

- **Before:** Circular dependencies can cause TC generation to hang or produce incomplete results
- **After:** Cycles detected early; user guided to resolve; valid stories still processed
- **Risk Level:** Reduces pipeline blockage from **MEDIUM** to **LOW**
