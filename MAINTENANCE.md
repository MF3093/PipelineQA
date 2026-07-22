# Maintenance & Sync Guide

This document explains how to keep **GitHub Copilot** and **Claude Code** agents in sync. Both systems run identical business logic — but in different formats with different tools.

---

## System Architecture

### Dual Structure

| Component | GitHub Copilot | Claude Code |
|-----------|---|---|
| **Agent definitions** | `.github/agents/*.agent.md` | `.claude/agents/*.md` |
| **Instructions** | `.github/instructions/*.instructions.md` | `.claude/instructions/*.md` |
| **Configuration** | `.vscode/mcp.json` | `.claude/settings.json` |
| **Tools** | read_file, grep_search, file_search, run_in_terminal, edit_file, create_file | Read, Grep, Glob, Bash, Edit, Write |

### Isolation

The two systems are **completely isolated**:
- Changes to `.github/` do NOT auto-sync to `.claude/`
- Changes to `.claude/` do NOT auto-sync to `.github/`
- Each system must be maintained separately

---

## What Must Stay in Sync

### 1. Agent Logic

If you update an agent in one system, update it in the other.

**Tool mapping:**
| Copilot | Claude Code |
|---------|---|
| `read_file` | `Read` |
| `grep_search` | `Grep` |
| `file_search` | `Glob` |
| `run_in_terminal` | `Bash` |
| `edit_file` | `Edit` |
| `create_file` | `Write` |

**Example:** If updating Fetcher in `.github/agents/fetcher.agent.md`, apply same changes to `.claude/agents/fetcher.md`, translating tools via the mapping above.

### 2. Global Rules (11 Rules)

**No merged copy exists.** Each system's instruction file is canonical for that system and must stay textually identical (content only — Copilot's file keeps its `applyTo` frontmatter; Claude's has none):

- `.github/instructions/global-rules.instructions.md` (Copilot)
- `.claude/instructions/global-rules.md` (Claude Code)

If you update a rule: edit both files in the same commit.

### 3. Path Schema (P-1 to P-8)

**No merged copy exists.** Same rule as above:

- `.github/instructions/path-schema.instructions.md` (Copilot)
- `.claude/instructions/path-schema.md` (Claude Code)

---

## What Can Diverge

✅ **Allowed differences:**

- Frontmatter format (syntax, structure)
- Tool names (read_file vs Read)
- File locations (.github/ vs .claude/)
- Command syntax (@agent vs /agent)
- Tool-specific conditional logic
- Platform-specific error handling

✅ **Example:** A Copilot agent can say "use `grep_search` for pattern matching" while the Claude Code agent says "use `Grep`" — this is OK because the tools are different. Business logic stays identical.

---

## When Adding Features

### Step 1: Design

- New business rule? Add to both `global-rules` instruction files (below)
- New path? Add a P-X rule to both `path-schema` instruction files
- Agent-specific? Document in the agent file only

### Step 2: Implement Copilot

Update:
- `.github/agents/{AGENT}.agent.md` (new logic)
- `.github/instructions/*.instructions.md` (if new rules/paths)

Test with GitHub Copilot. Commit.

### Step 3: Migrate to Claude Code

Update:
- `.claude/agents/{AGENT}.md` (copy logic from Copilot, map tools)
- `.claude/instructions/*.md` (same rules/paths, Claude syntax)

Test with Claude Code. Commit.

### Step 4: Document

Update:
- `QUICK-REFERENCE.md` (new agent, commands, or workflows)
- `GETTING-STARTED.md` (if user-facing workflow changed)
- `CLAUDE.md` / `.github/copilot-instructions.md` (if agent operating context changed)

---

## Sync Checklist

**Before each commit:**

```
[ ] Updated .github/agents/ and .claude/agents/ identically (tools translated)?
[ ] Updated .github/instructions/ and .claude/instructions/ with rule changes (both files, same content)?
[ ] Tool names correct in both systems?
[ ] Business logic identical between both versions?
[ ] Tested in both systems?
[ ] All cross-references valid?
[ ] No accidental modifications to unrelated files?
```

---

## Rollback Procedure

### If Claude Code Breaks

1. Revert `.claude/agents/{AGENT}.md` or `.claude/instructions/{FILE}.md`
2. Verify Copilot still works (use `.github/` as fallback)
3. Fix the issue in Claude version
4. Re-test and commit

### If Copilot Breaks

1. Revert `.github/agents/{AGENT}.agent.md` or `.github/instructions/{FILE}.instructions.md`
2. Verify Claude Code still works (use `.claude/` as fallback)
3. Fix the issue in Copilot version
4. Re-test and commit

---

## Validation Commands

```bash
# Verify agent count matches
ls .github/agents/*.agent.md | wc -l
ls .claude/agents/*.md | wc -l
# Should output the same number (8)

# Verify instruction files exist
test -f .github/instructions/global-rules.instructions.md && echo "Copilot rules OK"
test -f .claude/instructions/global-rules.md && echo "Claude rules OK"

# Check for accidental .github/ changes
git status .github/
```

---

## Common Sync Mistakes

| Mistake | Symptom | Fix |
|---------|---------|-----|
| Forgot to migrate Copilot changes to Claude | Claude agent doesn't have new feature | Copy logic to `.claude/agents/`, map tools, test |
| Updated rules in one system only | Rules inconsistent between systems | Edit both `global-rules` instruction files in the same commit |
| Changed tool names without docs | Users confused, docs stale | Update tool mapping table in `QUICK-REFERENCE.md` |
| Created paths not in schema | Agents write to wrong places | Add P-X rule to both `path-schema` instruction files before creating folders |

---

## Documentation Maintenance

Keep these in sync when rules or architecture changes:

| Document | Update When |
|----------|------------|
| `.github/instructions/*` + `.claude/instructions/*` | Rules or path schema change (edit both, identical content) |
| `QUICK-REFERENCE.md` | New agent, new commands, new workflows |
| `GETTING-STARTED.md` | New workflow, setup, or troubleshooting step affecting either system |
| `CLAUDE.md` | Agent roster/delegation changes affecting Claude Code |
| `.github/copilot-instructions.md` | Agent roster/delegation changes affecting Copilot |
| `MAINTENANCE.md` | Sync process changes, new patterns |

---

## See Also

- **Commands & syntax:** [QUICK-REFERENCE.md](QUICK-REFERENCE.md)
- **Getting started (both systems):** [GETTING-STARTED.md](GETTING-STARTED.md)
- **Claude Code agent context:** [CLAUDE.md](CLAUDE.md)
- **Copilot agent context:** [.github/copilot-instructions.md](.github/copilot-instructions.md)
- **Project overview:** [README.md](README.md)
