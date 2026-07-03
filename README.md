# PipelineQA — Multi-Agent System

A QA Test Case Generation Pipeline that operates as a multi-agent system. Each agent has a specific role with strictly scoped permissions. **Supported on both GitHub Copilot and Claude Code.**

## Platform Support

- **GitHub Copilot Chat** — Use agent picker (`@`) in VS Code
- **Claude Code** — Use slash commands (`/`) in Claude or command palette

See [CLAUDE.md](CLAUDE.md) for Claude Code setup and command reference.

## Quick Start

### Prerequisites
- **GitHub Copilot:** VS Code with GitHub Copilot Chat extension
- **Claude Code:** Claude with code editing support
- Jira MCP server configured (only required if your stories are in Jira — see GETTING-STARTED.md)

### Usage with GitHub Copilot

Select an agent from the agent picker (`@`) in Copilot Chat. The Orchestrator is the main entry point.

| Agent | Purpose |
|-------|---------|
| `@orchestrator` | Main entry point — coordinates the full pipeline |
| `@fetcher` | Reads stories from Jira/ADO and saves raw snapshots |
| `@parser` | Normalizes raw snapshots into structured ParsedStory JSON |
| `@story-analyzer` | Surfaces discrepancies and coverage gaps |
| `@context-builder` | Captures stable project-wide information |
| `@story-prioritizer` | Creates risk-based test prioritization matrix |
| `@tc-generator` | Generates precise, atomic test cases |
| `@tc-reviewer` | Cross-story TC analysis (standalone) |

### Pipeline Phases

1. **Phase 1 — Early Analysis**: Fetch → Parse → Story Analyze
2. **First Run Only — Project Context**: Context Build → Gate 2 approval
3. **Phase 2 — TC Generation**: Story Prioritizer → TC Generate

Key approval gates: **Gate 2** — project context, **Gate 3** — priority matrix, **Gate 4** — test cases per story (`yes` / `edit` / `reject` at each).

## Project Structure

```
.github/
├── copilot-instructions.md    ← Project-wide Copilot instructions
├── agents/                    ← Custom Copilot agents (.agent.md)
│   ├── orchestrator.agent.md
│   ├── fetcher.agent.md
│   ├── parser.agent.md
│   ├── story-analyzer.agent.md
│   ├── context-builder.agent.md
│   ├── story-prioritizer.agent.md
│   ├── tc-generator.agent.md
│   └── tc-reviewer.agent.md
├── instructions/              ← Auto-loaded rules (scoped to agents)
│   ├── global-rules.instructions.md
│   └── path-schema.instructions.md
└── skills/                    ← Cross-agent procedural specs
    ├── assumption-tracker.md
    └── prereq-checker.md

config/                        ← Templates for project configuration
docs/                          ← Testing documentation and runbooks
test-harness/                  ← Fixtures and test cases for agent validation
```

## Key Conventions

- All rules in `.github/instructions/global-rules.instructions.md` apply to every agent
- All file paths follow `.github/instructions/path-schema.instructions.md`
- Projects registered in `projects.json` with output paths
- Never invent content — log as assumption (`A-NNN`, `Q-NNN`, `B-NNN`, `D-NNN`)
- Approval gates use strict protocol: `yes` / `edit` / `reject`
- Never overwrite approved files without explicit permission

## Security

- All external inputs scanned for prompt injection (Rule 6)
- Agents have minimum permissions — no cross-boundary access
- Story sources are read-only — never modify source issues
