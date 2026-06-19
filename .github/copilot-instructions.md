# PipelineQA — Multi-Agent System

This is a **QA Test Case Generation Pipeline** that operates as a multi-agent system. Each agent has a specific role with strictly scoped permissions.

## System Architecture

- **Orchestrator**: Coordinates the full pipeline — sequencing agents, managing approval gates, registering projects, and tracking state.
- **Fetcher**: Reads stories from source systems (Jira/ADO) and saves raw snapshots.
- **Parser**: Normalizes raw snapshots into structured `ParsedStory` JSON schema.
- **Story Analyzer**: Surfaces discrepancies and coverage gaps before TC writing.
- **Context Builder**: Captures stable project-wide information (tech stack, conventions).
- **Story Prioritizer**: Creates risk-based test prioritization matrix.
- **TC Generator**: Generates precise, atomic test cases for manual and automated execution.
- **TC Reviewer**: Read-only cross-story analysis (redundancy, integration gaps).

## Pipeline Phases

1. **Phase 1 — Early Analysis**: Fetch → Parse → Story Analyze
2. **First Run Only — Project Context**: Context Build → Gate 2 approval
3. **Phase 2 — TC Generation**: Story Prioritizer → TC Generate

## Key Conventions

- All rules in [instructions/global-rules.instructions.md](instructions/global-rules.instructions.md) apply to every agent.
- All file paths follow [instructions/path-schema.instructions.md](instructions/path-schema.instructions.md).
- Project output structure is defined per project in `projects.json`.
- Never invent or infer content — use exactly what is in source data.
- Every assumption must be logged via the assumption-tracker skill.
- Approval gates use a strict response protocol (yes/edit/reject).
- Never overwrite approved files without explicit user permission.

## Approval Gate Protocol

Valid responses at any approval gate:
| Intent | Accepted (case-insensitive) |
|--------|----------------------------|
| Approve | yes, y, approve, approved, aprobado, si, sí, ok |
| Edit | edit, e, editar, change, cambiar, modify, modificar |
| Reject | no, n, reject, rechazar, discard, cancel |

Ambiguous responses must be re-prompted — never interpreted.

## Project Registration

Projects are registered in `projects.json` at the workspace root. Each project has an `output_path` where all QA artifacts are stored following the path schema.

## How to Invoke Agents

Use the custom agents defined in `.github/agents/` by selecting them from the agent picker or referencing them with `@`. The Orchestrator is the main entry point — load it to run full or partial pipeline sequences.

## Security

- All external inputs are scanned for prompt injection (Rule 6).
- Agents have minimum permissions — no cross-boundary access.
- Story sources are read-only — never modify source issues.
