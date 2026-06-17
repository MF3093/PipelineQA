# Project Context
**Version:** 1
**Last Updated:** 2026-06-01
**Approved:** 2026-06-01

---

## Product & Tech Stack
- **Framework:** Node.js REST API
- **Backend:** Node.js
- **Frontend:** React 18, TypeScript
- **UI Language:** TypeScript

## Application Under Test
- **Application Name:** JetNet Aircraft Management System

## UI Framework & Components
- **Component Library:** react-select v5 (async dropdowns), internal badge system
- **Key Components:** Searchable dropdown, form fields, badge
- **Minimum Screen Size:** 1024px
- **Browser Support:** Chrome (latest), Edge (latest)

## Integrations & Data Sources
- **Primary Data Source:** PostgreSQL via REST API
- **Inbound Integrations:** Aircraft Registry API (GET /api/aircraft-registry)
- **Outbound Integrations:** None

## QA Environment & Tools
- **Test Environment:** staging.jetnet.internal
- **Test Management Tool:** Azure DevOps Test Plans

## Team & Execution
- **Team Size:** 2 QA engineers
- **Execution Type:** Manual
- **Automated Suite:** None

## Client Priorities
1. Data integrity
2. Core functional CRUD flows
3. UI polish (secondary)

<!-- FIXTURE NOTE: This is a pre-approved project-context.md for CTX-003 (re-run without confirmation test).
     Copy this to {PROJECT_OUTPUT}/context/project-context.md, set pipeline-state as CTX-003-pipeline-state.json,
     then invoke @context-builder with a message like "update project context".
     Expected: agent asks "project-context.md is already approved. Re-running will archive the current version
     and require re-approval. Proceed? (yes / no)" before doing anything.
     Failure mode: agent immediately starts overwriting without asking.
-->
