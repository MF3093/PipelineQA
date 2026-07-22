# Claude Code Agents & Instructions

This directory contains all Claude Code agents and instructions for the PipelineQA multi-agent QA system.

## Quick Start

### View All Agents

See `.claude/agents/` directory for 8 agents:
- `orchestrator.md` - Pipeline coordinator
- `fetcher.md` - Story fetcher from Jira/ADO
- `parser.md` - Story normalizer
- `story-analyzer.md` - Discrepancy detector
- `context-builder.md` - Project context builder
- `story-prioritizer.md` - Risk-based prioritization
- `tc-generator.md` - Test case generator
- `tc-reviewer.md` - Cross-story analyzer

### View Instructions

See `.claude/instructions/` directory for:
- `global-rules.md` - 11 global rules applying to all agents
- `path-schema.md` - Standard folder structure and path rules

### Use Slash Commands

```
/orchestrator        # Run pipeline
/parser TEST-1       # Parse stories
/fetcher             # Fetch from Jira/ADO
/story-analyzer      # Analyze discrepancies
/context-builder     # Build project context
/story-prioritizer   # Create priority matrix
/tc-generator        # Generate test cases
/tc-reviewer         # Review cross-story
```

## Migration Information

This system was migrated from GitHub Copilot to Claude Code on 2026-07-21.

See these files for details:
- `MIGRATION_SUMMARY.md` - Quick overview
- `MIGRATION_LOG.md` - Detailed migration log
- `VERIFICATION_CHECKLIST.md` - Verification steps
- `../MIGRATION_REPORT.md` - Complete migration report

## Structure

```
.claude/
├── agents/
│   ├── context-builder.md
│   ├── fetcher.md
│   ├── orchestrator.md
│   ├── parser.md
│   ├── story-analyzer.md
│   ├── story-prioritizer.md
│   ├── tc-generator.md
│   └── tc-reviewer.md
├── instructions/
│   ├── global-rules.md
│   └── path-schema.md
├── commands/
│   └── [8 command files for slash commands]
└── README.md (this file)
```

## Original System Preserved

All original GitHub Copilot agents remain unchanged in `.github/agents/` and `.github/instructions/`. No destructive operations were performed.

## Testing

Start with individual agent testing:
```
/orchestrator
/parser TEST-STORY-1
/fetcher
```

Then proceed to Phase 1 and Phase 2 full pipeline tests.

See `MIGRATION_REPORT.md` for complete testing recommendations.

## Support

- **Agent details:** See individual `.claude/agents/*.md` files
- **Rules:** See `.claude/instructions/global-rules.md`
- **Paths:** See `.claude/instructions/path-schema.md`
- **Commands:** See `.claude/commands/*.md` for usage examples

---

Migration completed 2026-07-21. System ready for testing.
