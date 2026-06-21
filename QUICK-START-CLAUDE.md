# Quick Start — Running PipelineQA in Claude Code

## ✅ Setup Complete

Your PipelineQA agents have been migrated from GitHub Copilot to Claude Code. Everything is ready to run.

---

## Get Started in 3 Steps

### Step 1: Restart Claude Code

Close and reopen Claude Code with the PipelineQA workspace. It will automatically load `CLAUDE.md` and make all rules and configuration available.

### Step 2: Start the Orchestrator

Open the Claude Code chat and type:

```
/orchestrator
```

This is the main entry point. It will:
- Ask you to select a project (TEST-HARNESS-EPICS or TEST-HARNESS-NO-EPICS)
- Present the run menu with options for Phase 1, Phase 2, or individual agents

### Step 3: Execute Your Test

Choose a phase and follow the prompts. For example, to test the Parser:

```
/parser TEST-FMT-001
```

---

## Available Commands

All 8 agents are available as slash commands:

| Command | What it does |
|---|---|
| `/orchestrator` | Coordinate the full pipeline (main entry point) |
| `/fetcher STORY-KEY [...]` | Fetch stories from Jira/ADO |
| `/parser STORY-KEY [...]` | Parse raw story JSON to structured format |
| `/story-analyzer STORY-KEY` | Analyze stories for discrepancies and gaps |
| `/context-builder` | Build project-wide context (tech stack, conventions) |
| `/story-prioritizer STORY-KEY [...]` | Create risk-based prioritization matrix |
| `/tc-generator STORY-KEY` | Generate test cases |
| `/tc-reviewer [cross-story\|integration\|both]` | Review TCs for redundancy and conflicts |

---

## Example: Test the Parser

The FMT (Format) tests are ready to run. The fixtures have been set up in the project:

```
/parser TEST-FMT-001
```

Expected output:
- Parser reads `stories/raw/TEST-FMT-001.raw.json`
- Parses the story (detects format, extracts ACs, sections, etc.)
- Writes `stories/parsed/TEST-FMT-001.parsed.json`
- Updates registry with parsed status

Then document the result in `docs/FMT-TEST-RESULTS.md`.

---

## What Changed from Copilot

### Before (Copilot)
```
@parser
Parse story TEST-FMT-001
```

### Now (Claude Code)
```
/parser TEST-FMT-001
```

Same instructions, same logic, same outputs. Just different tool names under the hood.

---

## Key Information Loaded Automatically

When you use any command, Claude Code automatically has:

✅ **All 10 Global Rules** — from CLAUDE.md (Rules 1–10)  
✅ **Path Schema** — folder layout and file structure rules  
✅ **Tool Mapping** — Copilot tool names → Claude Code tools  
✅ **Approval Gate Protocol** — strict response format (yes/edit/reject)  
✅ **Project Configuration** — where to find project output folders  

You don't need to paste anything or configure anything — it's all in `CLAUDE.md`.

---

## Tool Mapping (Automatic)

You don't need to think about this, but here's what's happening under the hood:

| Original (Copilot) | Claude Code |
|---|---|
| `read_file` → `Read` tool |
| `grep_search` → `Grep` tool |
| `file_search` → `Glob` tool |
| `run_in_terminal` → `Bash` tool |
| `edit_file` → `Edit` tool |
| `create_file` → `Write` tool |

---

## Running the FMT Format Coverage Tests

To execute all 11 format tests:

```
/parser TEST-FMT-001
/parser TEST-FMT-002
/parser TEST-FMT-003
/parser TEST-FMT-004
/parser TEST-FMT-005 TEST-FMT-001 TEST-FMT-004    # Mix batch
/parser TEST-FMT-006
/parser TEST-FMT-007
/parser TEST-FMT-008
/parser TEST-FMT-009
/parser TEST-FMT-010
/parser TEST-FMT-011
```

After each one:
1. Check that the output file was created in `stories/parsed/`
2. Compare against the assertions in `docs/FMT-TEST-RESULTS.md`
3. Record the result (✅ PASS or ❌ FAIL)

---

## Documentation

- **CLAUDE.md** — Main system guide (rules, path schema, tool mapping)
- **CLAUDE-MIGRATION-SUMMARY.md** — What was migrated and why
- **FMT-TEST-EXECUTION-GUIDE.md** — Step-by-step guide for format tests
- **FMT-TEST-RESULTS.md** — Template for documenting results
- **FMT-TESTS-NEXT-STEPS.md** — Your execution roadmap
- **.github/agents/*.agent.md** — Original agent definitions (unchanged)

---

## Troubleshooting

### "Command not found: /parser"

Make sure you're in the PipelineQA workspace and CLAUDE.md has been loaded. Restart Claude Code if needed.

### "CLAUDE.md not found"

The file should be at the workspace root. If missing, check that it was created:

```
ls -la CLAUDE.md
```

### Agent doesn't behave as expected

The agent is reading from `.github/agents/{agent}.agent.md`. If the behavior changed:

1. Check the agent definition file (e.g., `.github/agents/parser.agent.md`)
2. Verify that file hasn't been modified
3. Restart Claude Code to refresh the context

### Rule 10 violation: "Using Grep on registry files"

The agent should ONLY use the `Read` tool on:
- `fetched-stories.json`
- `fetched-epics.json`
- `pipeline-state.json`

Never use Grep or Glob on registry JSON files. If you see this error, the agent is violating Rule 10 and needs to be corrected.

---

## Next Steps

1. ✅ **Migration complete** — all files created and verified
2. ⏳ **Run the FMT tests** — `/parser TEST-FMT-001` through `/parser TEST-FMT-011`
3. ⏳ **Document results** — fill in FMT-TEST-RESULTS.md with actual vs. expected
4. ⏳ **Update adversarial-testing.md** — add Category 14 with FMT results
5. ⏳ **Close the Copilot limit issue** — you now have unlimited agent runs in Claude Code

---

## Questions?

Refer to:
- **CLAUDE.md** — System rules and conventions
- **`.github/agents/{agent}.agent.md`** — Original agent specifications
- **`docs/` folder** — All QA test documentation

Everything you need is already in the project. The agents are ready to run.

**Good luck! 🚀**
