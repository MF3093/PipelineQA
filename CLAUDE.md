# PipelineQA — Agent Operating Context

Multi-agent QA test case generation pipeline. You are one of 8 agents operating under strict scope and rule constraints.

@.claude/instructions/global-rules.md
@.claude/instructions/path-schema.md

## Agent Roster & Delegation

| Agent | Invoke | Role |
|-------|--------|------|
| Orchestrator | `/orchestrator` | Entry point — sequencing, gates, project registry, state |
| Fetcher | `/fetcher STORY-KEY` | Only agent touching the story source (Jira/ADO) |
| Parser | `/parser STORY-KEY` | Raw snapshot → `ParsedStory` schema |
| Story Analyzer | `/story-analyzer STORY-KEY` | Discrepancy/gap detection, no TC writing |
| Context Builder | `/context-builder` | First-run project context only |
| Story Prioritizer | `/story-prioritizer` | Risk-based priority matrix |
| TC Generator | `/tc-generator STORY-KEY` | Test case generation |
| TC Reviewer | `/tc-reviewer` | Read-only cross-story review |

Pipeline order: Fetch → Parse → Story Analyze → [Context Build, first run] → Prioritize → TC Generate → [TC Review, optional].

Definitions: `.claude/agents/{name}.md`. Command wiring: `.claude/commands/{name}.md`.

## Scope Discipline

Each agent operates only within its own definition file's declared inputs/outputs. Do not read or write outside that scope even if a file is technically reachable. When unsure, stop and report to the Orchestrator rather than guessing.

## Cross-Reference

Human-facing setup, workflows, and troubleshooting: [GETTING-STARTED.md](GETTING-STARTED.md), [QUICK-REFERENCE.md](QUICK-REFERENCE.md). GitHub Copilot equivalent of this file: [.github/copilot-instructions.md](.github/copilot-instructions.md) + `.github/instructions/`.
