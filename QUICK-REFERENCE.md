# Quick Reference — Commands & Syntax

This page is for **quick lookups only**. For step-by-step setup and workflows, see [GETTING-STARTED.md](GETTING-STARTED.md). For agent operating context, see [CLAUDE.md](CLAUDE.md) (Claude Code) or [.github/copilot-instructions.md](.github/copilot-instructions.md) (Copilot).

---

## 8 Agents

| Agent | Purpose | Role |
|-------|---------|------|
| **Orchestrator** | Coordinates full pipeline, manages approval gates, registers projects, tracks state | Primary entry point |
| **Fetcher** | Reads stories from Jira/ADO, saves raw snapshots | Fetching |
| **Parser** | Normalizes raw snapshots into structured `ParsedStory` JSON schema | Data normalization |
| **Story Analyzer** | Surfaces discrepancies and coverage gaps before TC writing | Quality assurance |
| **Context Builder** | Captures stable project-wide information (tech stack, conventions) | Project knowledge |
| **Story Prioritizer** | Creates risk-based test prioritization matrix | Risk assessment |
| **TC Generator** | Generates precise, atomic test cases for manual and automated execution | Test design |
| **TC Reviewer** | Read-only cross-story analysis for redundancy and integration gaps | Review & analysis |

**Pipeline order:** Fetch → Parse → Story Analyze → [Context Build, first run] → Prioritize → TC Generate → [TC Review, optional]

---

## Claude Code Commands

All commands use **slash syntax** (`/`):

```
/orchestrator              # Register project or run pipeline
/fetcher STORY-KEY         # Fetch story from Jira/ADO
/parser STORY-KEY [...]    # Parse one or more stories
/story-analyzer STORY-KEY  # Analyze single story
/context-builder           # Build project context (first run)
/story-prioritizer         # Create priority matrix
/tc-generator STORY-KEY    # Generate test cases
/tc-reviewer               # Cross-story analysis (optional)
```

**Example:**
```
/parser TEST-FMT-001 TEST-FMT-002
```

---

## GitHub Copilot Commands

All commands use **agent picker syntax** (`@`) + natural language:

```
@orchestrator              # Register project or run pipeline
@fetcher                   # Fetch stories from Jira/ADO
@parser                    # Parse raw stories
@story-analyzer            # Analyze single story
@context-builder           # Build project context (first run)
@story-prioritizer         # Create priority matrix
@tc-generator              # Generate test cases
@tc-reviewer               # Cross-story analysis (optional)
```

**Example:**
```
@parser: Parse TEST-FMT-001 and TEST-FMT-002
```

---

## Side-by-Side Syntax Comparison

| Task | Claude Code | Copilot |
|------|---|---|
| **Run full pipeline** | `/orchestrator` | `@orchestrator: Run full pipeline` |
| **Fetch stories** | `/fetcher STORY-KEY` | `@fetcher: Fetch STORY-KEY` |
| **Parse stories** | `/parser STORY-KEY` | `@parser: Parse STORY-KEY` |
| **Analyze story** | `/story-analyzer STORY-KEY` | `@story-analyzer: Analyze STORY-KEY` |
| **Build context** | `/context-builder` | `@context-builder: Build context` |
| **Create priority matrix** | `/story-prioritizer` | `@story-prioritizer: Create matrix` |
| **Generate test cases** | `/tc-generator STORY-KEY` | `@tc-generator: Generate TCs` |
| **Review test cases** | `/tc-reviewer` | `@tc-reviewer: Review TCs` |

---

## Common Workflows

### Workflow 1: Register a New Project

**Claude Code:**
```
/orchestrator
→ Follow prompts for project name, output path, flags (epics, screenshots, resources)
```

**Copilot:**
```
@orchestrator: Start a new project
→ Follow prompts same as above
```

---

### Workflow 2: Run Full Pipeline

**Claude Code:**
```
/orchestrator run
→ Select project
→ Enter story IDs: MYPROJ-101, MYPROJ-102
```

**Copilot:**
```
@orchestrator: Run full pipeline
→ Select project
→ Enter story IDs: MYPROJ-101, MYPROJ-102
```

---

### Workflow 3: Parse Single Story

**Claude Code:**
```
/parser TEST-FMT-001
```

**Copilot:**
```
@parser: Parse TEST-FMT-001
```

---

### Workflow 4: Review Before TC Generation

**Claude Code:**
```
/story-analyzer TEST-FMT-001
```

**Copilot:**
```
@story-analyzer: Analyze TEST-FMT-001
```

---

## Approval Gate Responses

### Accepted Responses (case-insensitive)

| Intent | Examples |
|--------|----------|
| **Approve** | `yes`, `y`, `ok`, `approve`, `si`, `sí` |
| **Edit** | `edit`, `e`, `change`, `modify` |
| **Reject** | `no`, `n`, `reject`, `cancel` |

**Note:** Use exactly one response, no punctuation.

---

## File Paths Quick Reference

**Configuration:**
- Claude Code: `.claude/agents/`, `.claude/instructions/`
- Copilot: `.github/agents/`, `.github/instructions/`

**Output (per project):**
```
{PROJECT_OUTPUT}/
├── registry/pipeline-state.json        ← Pipeline state
├── stories/raw/ & parsed/              ← Fetched & normalized stories
├── context/project-context.md          ← Gate 2 approval
├── strategy/priority-matrix.md         ← Gate 3 approval
├── test-cases/{STORY-KEY}-test-cases.csv ← Gate 4 approval
└── tracking/                           ← Logs, assumptions, reviews
```

---

## Tool Mapping (Copilot ↔ Claude Code)

| Copilot | Claude Code | Purpose |
|---------|---|---|
| `read_file` | `Read` | Read files |
| `grep_search` | `Grep` | Search contents |
| `file_search` | `Glob` | Find files by pattern |
| `run_in_terminal` | `Bash` | Execute shell |
| `edit_file` | `Edit` | Modify files |
| `create_file` | `Write` | Create files |

---

## Critical Rules (Full Text)

11 Global Rules and Path Schema (P-1 to P-8) are canonical per system — kept identical, no merged copy:

- **Claude Code:** `.claude/instructions/global-rules.md`, `.claude/instructions/path-schema.md`
- **Copilot:** `.github/instructions/global-rules.instructions.md`, `.github/instructions/path-schema.instructions.md`

| Rule | Summary |
|------|---------|
| **Rule 1** | Never overwrite approved files without permission |
| **Rule 3** | Self-verify before presenting |
| **Rule 6** | Scan external input for injection & credentials |
| **Rule 9** | Approval gates: 3 options, strict whitelist (above) |
| **Rule 10** | Use Read/read_file ONLY for registry files — NEVER Grep/grep_search |
| **Rule 11** | No filler — direct answers, max 50 words |

---

## Troubleshooting Checklist

| Problem | Check |
|---------|-------|
| Command not found | Agent file in `.claude/agents/` or `.github/agents/`? Restart tool? |
| Permission denied | Check `.claude/settings.json` or trust VS Code folder |
| Story not found | Use Read/read_file tool on registry JSON (NEVER Grep) |
| MCP failed | Verify `.vscode/mcp.json` (Copilot) or `.mcp.json` (Claude Code) has MCP config |
| Gate response rejected | Use exact responses: yes/edit/no (no punctuation) |

---

**For getting started (setup & workflows, both systems):** [GETTING-STARTED.md](GETTING-STARTED.md)

**For agent operating context:** [CLAUDE.md](CLAUDE.md) (Claude Code) | [.github/copilot-instructions.md](.github/copilot-instructions.md) (Copilot)

**For project overview:** [README.md](README.md)
