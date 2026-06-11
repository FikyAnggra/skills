# QA-Reporter Implementation Plan

Date: 2026-06-11
Spec: `docs/superpowers/specs/2026-06-11-qa-reporter-design.md`
Status: Draft for implementation approval

## Goal

Build a portable `qa-reporter` agent skill package that converts planner data, executor results, manual execution results, or exploratory issue findings into governed reporting packages.

The package must follow the same portable plugin/skill pattern used by `qa-planner`.

## Target Structure

```text
qa-reporter/
  .codex-plugin/
    plugin.json
  .claude-plugin/
    plugin.json
  skills/
    qa-reporter/
      SKILL.md
      schemas/
        report-state.schema.json
        issue-submission-package.schema.json
        reporter-review.schema.json
        signoff-recommendation.schema.json
      templates/
        test-report.md
        issue-package.md
        bug-report.md
        sit-document.md
        uat-document.md
        coverage-risk-summary.md
        report-state.json
      examples/
        manual-execution-result.md
        exploratory-issue.md
        report-state.example.json
        output-test-report.example.md
        output-issue-package.example.md
      agents/
        openai.yaml
  README.md
  CHANGELOG.md
```

## Work Items

### 1. Create Plugin and Skill Skeleton

Create `qa-reporter/` plugin wrapper and `skills/qa-reporter/` portable skill.

Acceptance criteria:
- `.codex-plugin/plugin.json` exists.
- `.claude-plugin/plugin.json` exists.
- `skills/qa-reporter/SKILL.md` exists.
- Package is self-contained.
- `CHANGELOG.md` exists and records initial package version.
- Core skill instructions stay platform-neutral.

### 2. Write `SKILL.md`

Create portable reporter instructions.

Content must include:
- purpose and triggers
- three input paths: planner+executor, manual execution result, exploratory issue finding
- `report_state.json` source of truth
- report metric calculation rules
- issue package governance
- severity normalization
- SIT/UAT document generation
- sign-off recommendation rules
- multi-level OK/NOK review loop
- output formats
- issue submission gate
- quality gates

Acceptance criteria:
- Skill can be read standalone by another agent.
- Skill does not tell reporter to run tests, create test cases, or write automation scripts.
- Skill does not submit issue final unless user explicitly asks and package/config are approved.

### 3. Add JSON Schemas

Create:
- `report-state.schema.json`
- `issue-submission-package.schema.json`
- `reporter-review.schema.json`
- `signoff-recommendation.schema.json`

Schema rules:
- `report-state.schema.json` supports canonical state, metrics, issue package, SIT/UAT sections, review history, submission history, changelog.
- `issue-submission-package.schema.json` supports normalized issue package, redaction status, submission status, and submission result.
- `reporter-review.schema.json` supports multi-level OK/NOK feedback.
- `signoff-recommendation.schema.json` supports `Go`, `Conditional Go`, `No-Go`, and `Not Assessed`, including override metadata.

Acceptance criteria:
- All schema files are valid JSON.
- Enums match approved design exactly.
- Extension fields are controlled through `custom_fields` where needed.

### 4. Add Templates

Create:
- `test-report.md`
- `issue-package.md`
- `bug-report.md`
- `sit-document.md`
- `uat-document.md`
- `coverage-risk-summary.md`
- `report-state.json`

Acceptance criteria:
- Templates render from `report_state`.
- Test report includes execution summary, metrics, failed/blocked details, issue summary, evidence refs.
- Issue package template is review/submission-ready.
- Bug report supports exploratory finding path.
- SIT/UAT templates include sign-off recommendation and reason.
- Coverage/risk summary cites source refs when available.

### 5. Add Examples

Create:
- `manual-execution-result.md`
- `exploratory-issue.md`
- `report-state.example.json`
- `output-test-report.example.md`
- `output-issue-package.example.md`

Acceptance criteria:
- Example includes planner/executor style reporting input.
- Example includes manual execution result path.
- Example includes exploratory issue path.
- Example includes at least one issue package.
- Example includes one sign-off recommendation with reason.
- Example shows report gaps where evidence/data is missing.

### 6. Add README

Create `README.md` and `CHANGELOG.md` for humans installing, reviewing, and tracking package changes.

Content must include:
- what `qa-reporter` does
- package contents
- three input paths
- expected outputs
- sign-off recommendation model
- review loop
- issue submission gate
- downstream workflow summary

Acceptance criteria:
- README is short but complete.
- README does not assume one specific agent platform.
- `CHANGELOG.md` records initial version, compatibility targets, schema/template additions, and future migration notes.

### 7. Generate Skill UI Metadata

Create `agents/openai.yaml` for Codex-compatible display metadata.

Acceptance criteria:
- File exists under `skills/qa-reporter/agents/openai.yaml`.
- Metadata matches `SKILL.md` behavior.

### 8. Validate Package

Run local checks:
- Confirm required file structure.
- Parse all JSON files.
- Check `SKILL.md` frontmatter has name and description.
- Search unresolved placeholder markers.
- Confirm Codex plugin manifest exists and parses.
- Confirm Claude plugin manifest exists and parses.
- Confirm `CHANGELOG.md` exists and has an initial entry.
- Run official validator scripts if runtime dependencies allow it.

Acceptance criteria:
- No invalid JSON.
- No unresolved placeholder markers.
- All referenced files exist.
- Validator failures caused by missing runtime dependencies are documented.

## Implementation Order

1. Create Codex and Claude plugin scaffolds.
2. Initialize portable skill scaffold.
3. Write `SKILL.md`.
4. Write schemas.
5. Write templates.
6. Write examples.
7. Write `README.md` and `CHANGELOG.md`.
8. Generate `agents/openai.yaml`.
9. Validate.
10. Summarize generated package and limitations.

## Risks

- Report metrics may conflict with manual input. Mitigation: keep source refs and report gaps visible.
- Issue severity may be subjective. Mitigation: record normalization inputs and allow human override.
- SIT/UAT recommendation may be disputed. Mitigation: show rule, reason, and override metadata.
- Issue tracker tooling may be unavailable. Mitigation: generate approved issue submission package without submitting.
- Evidence may contain secrets. Mitigation: enforce redaction status before submission.

## Done Criteria

Implementation is done when:
- `qa-reporter/` package exists.
- All target files exist.
- `.codex-plugin/plugin.json` exists and parses.
- `.claude-plugin/plugin.json` exists and parses.
- `CHANGELOG.md` exists with an initial entry.
- JSON files parse correctly.
- Package contains no unresolved placeholder markers.
- `SKILL.md` frontmatter is valid.
- README explains all three input paths and issue submission gate.
- Final summary lists created files and validation result.
