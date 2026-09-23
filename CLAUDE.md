# PipelineQA — Agent Operating Context

Multi-agent QA test case generation pipeline. You are one of 8 agents operating under strict scope and rule constraints.

@.claude/instructions/global-rules.md
@.claude/instructions/path-schema.md

## Agent Roster & Delegation

| Agent | `subagent_type` | Role |
|-------|-----------------|------|
| Orchestrator | `orchestrator` | Entry point — sequencing, gates, project registry, state |
| Fetcher | `fetcher` | Only agent touching the story source (Jira/ADO) |
| Parser | `parser` | Raw snapshot → `ParsedStory` schema |
| Story Analyzer | `story-analyzer` | Discrepancy/gap detection, no TC writing |
| Context Builder | `context-builder` | First-run project context only |
| Story Prioritizer | `story-prioritizer` | Risk-based priority matrix |
| TC Generator | `tc-generator` | Test case generation |
| TC Reviewer | `tc-reviewer` | Read-only cross-story review |

Pipeline order: Fetch → Parse → Story Analyze → [Context Build, first run] → Prioritize → TC Generate → [TC Review, optional].

Definitions: `.claude/agents/{name}.md` — these are the single source of truth. There is no
separate command-wiring layer.

## Invocation

Every agent is a subagent, launched through the Agent tool with its name as `subagent_type`.
There are no slash commands for these agents. Two entry points:

- **Full pipeline** — invoke `orchestrator`. It resolves the project, manages every approval
  gate, and delegates to the other agents in order, passing `{PROJECT_OUTPUT}` and the target
  story keys to each one.
- **Single agent (standalone)** — invoke any agent directly, e.g. "run the `parser` agent on
  TEST-001". Each agent's own **Invocation** section defines how it resolves `{PROJECT_OUTPUT}`
  from `projects.json` and how it asks for a target when none is given. Standalone runs
  bypass the Orchestrator's gates and state tracking — use them for targeted re-runs and
  debugging, not for a full pipeline pass.

## Scope Discipline

Each agent operates only within its own definition file's declared inputs/outputs. Do not read or write outside that scope even if a file is technically reachable. When unsure, stop and report to the Orchestrator rather than guessing.

## Cross-Reference

Human-facing setup, workflows, and troubleshooting: [GETTING-STARTED.md](GETTING-STARTED.md), [QUICK-REFERENCE.md](QUICK-REFERENCE.md). GitHub Copilot equivalent of this file: [.github/copilot-instructions.md](.github/copilot-instructions.md) + `.github/instructions/`.
