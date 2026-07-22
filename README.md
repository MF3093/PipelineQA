# PipelineQA — Multi-Agent QA System

A QA Test Case Generation Pipeline that operates as a multi-agent system. Each agent has a specific role with strictly scoped permissions. **Supported on both GitHub Copilot and Claude Code.**

---

## Documentation

Five human-facing docs cover this project; `CLAUDE.md` and `.github/copilot-instructions.md` are minimal agent operating context, not human guides.

### Quick Links

| For... | Read this |
|--------|-----------|
| New users (setup & workflows, both systems) | [GETTING-STARTED.md](GETTING-STARTED.md) |
| Agent roster, commands, syntax (both systems) | [QUICK-REFERENCE.md](QUICK-REFERENCE.md) |
| Keeping both systems in sync | [MAINTENANCE.md](MAINTENANCE.md) |
| Claude Code agent operating context | [CLAUDE.md](CLAUDE.md) |
| Copilot agent operating context | [.github/copilot-instructions.md](.github/copilot-instructions.md) |

---

## Quick Overview

### 8 Agents

Orchestrator • Fetcher • Parser • Story Analyzer • Context Builder • Story Prioritizer • TC Generator • TC Reviewer

See [QUICK-REFERENCE.md — 8 Agents](QUICK-REFERENCE.md#8-agents) for descriptions.

### 3 Phases

1. **Phase 1** — Fetch → Parse → Story Analyze
2. **Phase 2** — Context Build (first run only)
3. **Phase 3** — Prioritize → Generate Test Cases

### Platform Support

- **GitHub Copilot Chat** — Agent picker (`@`) in VS Code → See [GETTING-STARTED.md](GETTING-STARTED.md)
- **Claude Code** — Slash commands (`/`) → See [GETTING-STARTED.md](GETTING-STARTED.md)

---

## Project Structure

```
.github/agents/                     ← Copilot agent definitions
.github/instructions/               ← Global rules & path schema (Copilot)
.claude/agents/                     ← Claude Code agent definitions
.claude/instructions/               ← Global rules & path schema (Claude Code)
projects.json                       ← Project registry (output paths)
GETTING-STARTED.md                  ← Step-by-step setup guide (both systems)
QUICK-REFERENCE.md                  ← Agents, commands, syntax (both systems)
CLAUDE.md                           ← Claude Code agent operating context
.github/copilot-instructions.md     ← Copilot agent operating context
MAINTENANCE.md                      ← Sync & maintenance
```

---

## System Principles

- **11 Global Rules** — Applied consistently to all agents (`.claude/instructions/global-rules.md` / `.github/instructions/global-rules.instructions.md`)
- **8 Path Rules (P-1 to P-8)** — Define folder structure (`.claude/instructions/path-schema.md` / `.github/instructions/path-schema.instructions.md`)
- **Strict Approval Gates** — 3 options per gate, whitelisted responses only
- **Minimum Permissions** — Each agent touches only assigned files
- **Security First** — Prompt injection & credential scanning on all external input
- **Dual Maintenance** — Both Copilot and Claude Code receive identical updates

---

## Getting Started

1. **First time?** → [GETTING-STARTED.md](GETTING-STARTED.md)
2. **Need agents/commands?** → [QUICK-REFERENCE.md](QUICK-REFERENCE.md)
3. **Keeping systems in sync?** → [MAINTENANCE.md](MAINTENANCE.md)

---

## FAQ

**Where do I start if I'm new?**
[GETTING-STARTED.md](GETTING-STARTED.md).

**How do I invoke agents in Claude Code vs Copilot?**
[QUICK-REFERENCE.md — Claude Code Commands](QUICK-REFERENCE.md#claude-code-commands) and [QUICK-REFERENCE.md — GitHub Copilot Commands](QUICK-REFERENCE.md#github-copilot-commands).

**What are the approval gates?**
4 gates: Gate 1 (fetched), Gate 2 (context), Gate 3 (priority), Gate 4 (test cases). See [QUICK-REFERENCE.md — Approval Gate Responses](QUICK-REFERENCE.md#approval-gate-responses).

**How do I keep Copilot and Claude Code in sync?**
[MAINTENANCE.md](MAINTENANCE.md).
