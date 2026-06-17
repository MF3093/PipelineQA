# Architecture — QA Pipeline Multi-Agent System

This document provides a visual overview of the pipeline's agent orchestration, data flow, and governance model.

---

## System Overview

```mermaid
graph TB
    subgraph User["👤 User"]
        U[Commands & Approvals]
    end

    subgraph Orchestrator["Orchestrator (Coordinator)"]
        O[Sequencing · Gates · State · Logging]
    end

    subgraph Phase1["Phase 1 — Early Analysis"]
        F[Fetcher]
        P[Parser]
        SA[Story Analyzer]
    end

    subgraph Phase2["Phase 2 — TC Generation"]
        CB[Context Builder]
        PR[Prioritizer]
        TG[TC Generator]
    end

    subgraph Standalone["Standalone Agents"]
        TR[TC Reviewer]
        BR[Bug Reporter]
    end

    subgraph External["External Systems"]
        JIRA[Jira / ADO / GitHub]
    end

    U -->|"load orchestrator"| O
    O -->|"invoke"| F
    O -->|"invoke"| P
    O -->|"invoke"| SA
    O -->|"invoke"| CB
    O -->|"invoke"| PR
    O -->|"invoke"| TG
    U -->|"load directly"| TR
    U -->|"load directly"| BR

    F -->|"MCP read-only"| JIRA
    BR -->|"create issue + link"| JIRA

    style O fill:#2d5a87,color:#fff
    style F fill:#4a7c59,color:#fff
    style P fill:#4a7c59,color:#fff
    style SA fill:#4a7c59,color:#fff
    style CB fill:#7c5a2d,color:#fff
    style PR fill:#7c5a2d,color:#fff
    style TG fill:#7c5a2d,color:#fff
    style TR fill:#5a2d7c,color:#fff
    style BR fill:#5a2d7c,color:#fff
```

---

## Pipeline Sequence & Approval Gates

```mermaid
sequenceDiagram
    participant U as User
    participant O as Orchestrator
    participant F as Fetcher
    participant P as Parser
    participant SA as Story Analyzer
    participant CB as Context Builder
    participant PR as Prioritizer
    participant TG as TC Generator

    U->>O: Start run (Full / Phase 1 / Phase 2)
    O->>O: Lock pipeline, create run log

    rect rgb(230, 245, 230)
        Note over F,SA: Phase 1 — Early Analysis
        O->>F: Fetch stories by ID
        F-->>O: Raw snapshots saved (fetch summary shown)
        O->>P: Parse raw stories
        P-->>O: Parsed files saved
        O->>SA: Analyze stories
        SA-->>O: Q/D entries logged to assumptions.md
    end

    rect rgb(245, 237, 220)
        Note over CB,TG: Phase 2 — TC Generation
        O->>CB: Build context (first run only)
        CB-->>O: project-context.md drafted
        O->>U: Gate 2 — Approve context?
        U->>O: yes / edit / reject
        O->>O: Log gate decision + corrections
        O->>PR: Produce priority matrix
        PR-->>O: priority-matrix.md drafted
        O->>U: Gate 3 — Approve strategy?
        U->>O: yes / edit / reject
        O->>O: Log gate decision + corrections
        O->>TG: Generate TCs per story
        TG-->>O: TC files written
        O->>U: Gate 4 — Approve TCs?
        U->>O: yes / edit / reject
        O->>O: Log gate decision + corrections
    end

    O->>O: Archive resolved assumptions
    O->>O: Write run summary, release lock
    O->>U: Run complete — summary presented
```

---

## Data Flow & File Ownership

```mermaid
flowchart LR
    subgraph Sources["Input Sources"]
        JIRA[(Story Source<br/>Jira/ADO/GitHub)]
        SCREEN[Screenshots]
        EXTRA[ExtraResources]
        USER_INPUT[User Interview]
    end

    subgraph Agents["Processing Agents"]
        F[Fetcher]
        P[Parser]
        SA[Story Analyzer]
        CB[Context Builder]
        PR[Prioritizer]
        TG[TC Generator]
    end

    subgraph Outputs["Output Artifacts"]
        RAW[stories/raw/<br/>epics/raw/]
        PARSED[stories/parsed/<br/>epics/parsed/]
        CTX[context/<br/>project-context.md]
        STRAT[strategy/<br/>priority-matrix.md]
        TC[test-cases/<br/>*.csv]
        TRACK[tracking/<br/>assumptions.md]
        CORR[tracking/<br/>corrections-log.md]
        LOGS[tracking/logs/<br/>*.log.md]
    end

    JIRA --> F --> RAW
    RAW --> P --> PARSED
    PARSED --> SA --> TRACK
    SCREEN --> SA
    USER_INPUT --> CB --> CTX
    PARSED --> PR --> STRAT
    CTX --> PR
    PARSED --> TG --> TC
    STRAT --> TG
    SCREEN --> TG
    EXTRA --> TG
    TG --> TRACK
```

---

## Governance Model

```mermaid
flowchart TB
    subgraph Controls["Quality Controls"]
        G2[Gate 2: Context Approval]
        G3[Gate 3: Strategy Approval]
        G4[Gate 4: TC Approval]
    end

    subgraph Audit["Audit Trail"]
        RL[Run Log<br/>tracking/logs/RUN-ID.log.md]
        CL[Corrections Log<br/>tracking/corrections-log.md]
        AT[Assumptions<br/>tracking/assumptions.md]
        RH[Run History<br/>pipeline-state.json]
    end

    subgraph Rules["Enforcement Rules"]
        R1[Rule 1: Never overwrite approved]
        R2[Rule 2: Never invent information]
        R6[Rule 6: Prompt-injection defense]
        R7[Rule 7: Incremental only]
    end

    G2 --> RL
    G3 --> RL
    G4 --> RL
    G2 -->|"edit"| CL
    G3 -->|"edit"| CL
    G4 -->|"edit"| CL

    R1 --> G2
    R1 --> G3
    R1 --> G4
    R2 --> AT
    R6 -.->|"scan all external input"| Controls
```

---

## Permission Matrix

Each agent has strictly scoped read/write permissions. No agent operates outside its boundary.

| Agent | Reads | Writes | External Access |
|-------|-------|--------|-----------------|
| **Orchestrator** | pipeline-state, registries | pipeline-state, projects.json, run logs, corrections log | None |
| **Fetcher** | story source (MCP), registries, source-config | stories/raw, epics/raw, registries | Story source (read-only) |
| **Parser** | stories/raw, epics/raw | stories/parsed, epics/parsed | None |
| **Story Analyzer** | stories/parsed, epics/parsed, screenshots | tracking/assumptions.md | None |
| **Context Builder** | epics/parsed, stories/parsed, user input | context/project-context.md | None |
| **Prioritizer** | stories/parsed, epics/parsed, context | strategy/priority-matrix.md, tracking/assumptions.md | None |
| **TC Generator** | stories/parsed, context, strategy, screenshots, ExtraResources | test-cases/*.csv, tracking/assumptions.md | None |
| **TC Reviewer** | test-cases/*.csv, priority-matrix.md | tracking/reviews/ (optional) | None |
| **Bug Reporter** | TC files, registries, user input | bugs/drafts/ | Jira (create issue + link only) |

---

## Skills (Cross-Agent Utilities)

| Skill | Used By | Purpose |
|-------|---------|---------|
| **assumption-tracker** | Story Analyzer, Prioritizer, TC Generator, Context Builder | Log/resolve A-NNN, Q-NNN, B-NNN, D-NNN entries |
| **prereq-checker** | All agents (via Orchestrator) | Verify required files and states before execution |
