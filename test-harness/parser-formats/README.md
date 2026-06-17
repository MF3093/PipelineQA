# Parser Format Coverage Tests

Tests that the Parser produces a consistent, valid `ParsedStory` schema regardless of the
formatting style used in the story body. Covers the 4 variants found in this project plus 6 additional
formats encountered when the pipeline is used on other projects.

## Format Variants

| ID | File | Format Style | Real-world Examples |
|----|------|--------------|---------------------|
| FMT-001 | FMT-001.raw.json | Plain text `Section:` headers | Earliest batch stories (~14 stories) |
| FMT-002 | FMT-002.raw.json | Bold `**Section**` headers | Mid-batch stories (~16 stories) |
| FMT-003 | FMT-003.raw.json | Markdown `## Section` / `### Subsection` | Most detailed stories (~20 stories) |
| FMT-004 | FMT-004.raw.json | Informal / no structure (paragraph only) | H20-192, H20-288, H20-294 pattern |
| FMT-005 | (batch — use FMT-001 + FMT-004 together) | Mixed batch | Run both in a single parse session |
| FMT-006 | FMT-006.raw.json | User story triple ONLY — no AC section | Teams that write only "As a / I want / So that" |
| FMT-007 | FMT-007.raw.json | ACs as additional "As a / I want / So that" lines | Teams that skip numbered lists entirely |
| FMT-008 | FMT-008.raw.json | Gherkin / BDD — `Given / When / Then` scenario blocks | Cucumber, SpecFlow, Behave-based teams |
| FMT-009 | FMT-009.raw.json | Numbered list immediately after triple — no AC header label | Informal Jira writers skipping section headers |
| FMT-010 | FMT-010.raw.json | ADF JSON description (Jira Cloud new editor) | Any project on Jira Cloud |
| FMT-011 | FMT-011.raw.json | HTML description (Azure DevOps / older Jira) | ADO projects; Jira Server pre-2019 |

## How to Run

**Single format test (FMT-001, 002, 003, 004):**
1. Copy the fixture to `{PROJECT_OUTPUT}/stories/raw/`.
2. Add registry entry with `status: "fetched"`.
3. Load `@parser` and parse the story key.
4. Inspect the output `{STORY-KEY}.parsed.json` against the assertions below.

**Mixed batch test (FMT-005):**
1. Copy BOTH `FMT-001.raw.json` AND `FMT-004.raw.json` to `{PROJECT_OUTPUT}/stories/raw/`.
2. Add both registry entries with `status: "fetched"`.
3. Load `@parser` and parse both story keys in a single session.
4. Compare the two output files — schema shape must be identical.

## Key Assertions Per Format

### FMT-001 (Plain text `Section:`)
- `acs[]` populated with all 4 ACs, text copied exactly
- `sections[]` contains entries for "Specified Behavior" and "Dependencies" headers
- `out_of_scope[]` populated from "Out of Scope:" section
- `dependencies[]` populated from "Dependencies:" section
- **CRITICAL:** No ad-hoc top-level fields added (no `specified_behavior`, no `dependencies_text`)

### FMT-002 (Bold `**Section**`)
- Same schema shape as FMT-001
- `sections[]` captures "Specified Behavior" and "Technical Constraints" sections
- `acs[]` has 4 entries; `conditional: true` on ACs 3 and 4 (IF/not shown)
- **CRITICAL:** No ad-hoc top-level fields added (no `technical_constraints`)

### FMT-003 (Markdown `## Section` with subsections)
- `acs[]` has 5 entries; AC-1 has `conditional: false`
- `sections[]` captures "Specified Behavior" with subsections: "Field Behavior", "Entry Points", "Error Handling"
- Each subsection appears in `sections[N].subsections[]` with correct header and content
- `questions[]` has 2 entries; question 1 answered (comment resolves it), question 2 unanswered
- `comments[]` has 1 entry from "Product Owner"
- `dependencies[]` populated from "Dependencies" section

### FMT-004 (Informal / no structure)
- `description` = entire body text (no section headers found)
- `sections: []` (empty array — no extractable sections)
- `acs: []` (no AC section identifiable)
- `needs_review: true`
- `flags: ["NEEDS_REVIEW"]`
- **CRITICAL:** NO invented fields — no `requirements`, no `behavior_notes`, no `technical_details`

### FMT-005 (Mixed batch)
- Both output files must have the same top-level key set
- FMT-004 output has `acs: []` and `sections: []`; FMT-001 output has populated arrays
- Schema shape (key names present) is identical across both files

### FMT-006 (User story triple ONLY)
- `acs: []` — no ACs invented from the triple prose
- `needs_review: true`
- `flags` contains `"no_acs_found"`
- Parser must pause and surface the gap to the user — must NOT silently continue
- `description` = the triple sentence verbatim
- **CRITICAL:** No hallucinated ACs derived from the "So that" clause

### FMT-007 (ACs as additional "As a / I want / So that" lines)
- `acs[]` has 5 entries — one per triple line (excluding the opening framing triple)
- Each AC text = the full "As a ... I want ... so that ..." sentence
- `testable: true` on all 5 ACs (exportable feature request, not ambiguous)
- `needs_review: false` — content is present and extractable
- Schema shape identical to FMT-001
- **CRITICAL:** Lines must not be collapsed into a single AC or treated as narrative description

### FMT-008 (Gherkin / BDD — Given/When/Then)
- `acs[]` has 4 entries — one per Scenario block
- Each AC text = the full scenario block (Given/When/Then lines preserved)
- `testable: true` on all 4 ACs
- `sections: []` — no section headers present
- `needs_review: false`
- **CRITICAL:** Scenario blocks must not be flattened into a single AC or treated as prose

### FMT-009 (Numbered list, no AC header label)
- `acs[]` has 5 entries — one per numbered list item
- Parser infers the list is the AC section from context (immediately follows the user story triple)
- `needs_review: false` — content is present and extractable
- `sections: []` — no section headers present
- **CRITICAL:** No `"Acceptance Criteria"` key invented in `sections[]`; no items dropped

### FMT-010 (ADF JSON description — Jira Cloud)
- `acs[]` has 4 entries — extracted from the `orderedList` content nodes
- `out_of_scope[]` has 1 entry — extracted from the paragraph after the "Out of Scope" heading
- `description` = plain text extracted from the opening paragraph node
- **CRITICAL:** The raw ADF object must NOT appear anywhere in the parsed output
- No ad-hoc field `adf_content` or `raw_description` created
- Schema shape identical to FMT-001

### FMT-011 (HTML description — ADO / older Jira)
- `acs[]` has 4 entries — extracted from `<li>` items inside `<ol>`
- `out_of_scope[]` has 1 entry — extracted from `<p>` after `<h3>Out of Scope</h3>`
- `sections[]` has 1 entry for "Technical Constraints" — content extracted from final `<p>`
- `description` = plain text of opening `<p>` (HTML tags stripped)
- **CRITICAL:** No HTML tags in any field value — all output is plain text
- No ad-hoc field `html_body` or `raw_html` created
