# Architecture — PipelineQA

PipelineQA has no application code, server process, or deployed runtime. It is a set of
Markdown agent definitions (`.claude/agents/*.md` for Claude Code, `.github/agents/*.agent.md`
for GitHub Copilot) executed interactively inside the host tool, coordinated by the
Orchestrator agent and governed by a shared rules file
(`.claude/instructions/global-rules.md`). State and hand-offs between agents are files on
disk, not in-memory objects or API calls — every arrow in the diagram below is a file read
or write, per the authoritative layout in
[`.claude/instructions/path-schema.md`](../.claude/instructions/path-schema.md).

## Pipeline flow

```mermaid
flowchart TD
    USER([User]) -->|invokes| ORCH[Orchestrator]

    subgraph EXT["External boundary"]
        MCP[("Jira / ADO\nvia MCP")]
    end

    subgraph PHASE1["Phase 1 — Fetch → Parse → Analyze"]
        FETCH[Fetcher]
        PARSE[Parser]
        ANALYZE[Story Analyzer]
    end

    subgraph PHASE2["Phase 2 — Context Build (first run only)"]
        CTX[Context Builder]
    end

    subgraph PHASE3["Phase 3 — Prioritize → Generate → Review"]
        PRIO[Story Prioritizer]
        TCGEN[TC Generator]
        TCREV["TC Reviewer\n(optional, standalone)"]
    end

    REG[("registry/\npipeline-state.json\nfetched-stories.json\nfetched-epics.json")]

    ORCH -->|delegates, tracks state| REG
    ORCH --> FETCH
    ORCH --> PARSE
    ORCH --> ANALYZE
    ORCH -->|first run only| CTX
    ORCH --> PRIO
    ORCH --> TCGEN
    USER -.->|standalone, bypasses gates| TCREV

    FETCH <-->|only agent touching source| MCP
    FETCH -->|writes| RAW[("stories/raw/\nepics/raw/")]
    RAW -->|reads| PARSE
    PARSE -->|writes| PARSED[("stories/parsed/\nepics/parsed/")]
    PARSED -->|reads| ANALYZE
    PARSED -->|reads| CTX
    PARSED -->|reads| PRIO
    PARSED -->|reads| TCGEN

    ANALYZE -->|logs findings| ASSUMP[("tracking/assumptions.md")]
    CTX -->|writes, stable for project lifetime| PCTX[("context/project-context.md")]
    PCTX -->|reads| PRIO

    PRIO -->|writes| MATRIX[("strategy/priority-matrix.md\nstrategy/strategy-versions/")]
    MATRIX -->|reads, approved only| TCGEN
    PRIO -->|logs findings| ASSUMP

    TCGEN -->|writes| TC[("test-cases/{STORY}-test-cases.csv\ntest-data-requirements.md")]
    TCGEN -->|reads if present| SCR[("screenshots/, ExtraResources/\n(conditional on projects.json flags)")]
    TCGEN -->|logs findings| ASSUMP

    TC -->|reads, read-only| TCREV
    MATRIX -->|reads| TCREV
    ASSUMP -->|reads| TCREV
    TCREV -->|writes, non-blocking| REVOUT[("tracking/reviews/")]
```

## Notes

- **Gates.** Every phase transition in the Orchestrator's delegated flow requires explicit
  user approval (`global-rules.md` Rule 9) before the next agent runs — the diagram shows
  data flow, not an autonomous loop.
- **Single external boundary.** Only the Fetcher talks to Jira/ADO (via the
  `mcp__claude_ai_Atlassian_Rovo__getJiraIssue` MCP tool). Every other agent reads locked
  snapshots the Fetcher already wrote — no other agent can reach the story source.
- **Conditional paths.** `epics/`, `screenshots/`, and `ExtraResources/` only exist when the
  corresponding flag (`has_epics`, `has_screenshots`, `has_extra_resources`) is set per
  project in `projects.json`; see `path-schema.md` for the full conditional rules (P-3 to P-5).
- **TC Reviewer is out-of-band.** It runs standalone on user request, reads already-approved
  artifacts, and only ever writes recommendation reports — it never blocks the pipeline and
  is not sequenced by the Orchestrator.
- **Two platform trees, one contract.** `.github/agents/*.agent.md` (Copilot) mirrors
  `.claude/agents/*.md` (Claude Code) with identical roles, inputs, and outputs; the one
  documented deviation is the Fetcher invocation path (inline vs. subagent) noted in
  `orchestrator.md`, which is a platform capability difference, not a business-logic change.
