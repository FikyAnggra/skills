# QA-Planner Implementation Plan

Date: 2026-06-10
Spec: `docs/superpowers/specs/2026-06-10-qa-planner-design.md`
Status: Draft for implementation approval

## Goal

Build a portable `qa-planner` agent skill package that can be installed or copied across agent platforms. The package must define platform-neutral behavior, schemas, templates, examples, and validation guidance for QA planning workflows.

## Target Structure

```text
qa-planner/
  .codex-plugin/
    plugin.json
  .claude-plugin/
    plugin.json
  skills/
    qa-planner/
      SKILL.md
      schemas/
        planning-state.schema.json
        handoff-contract.schema.json
        review-feedback.schema.json
      templates/
        Template Test Case.xlsx
        test-plan.md
        test-case.md
        handoff-contract.json
      examples/
        input-requirement.md
        planning-state.example.json
        output-test-cases.example.md
      agents/
        openai.yaml
  README.md
  CHANGELOG.md
```

## Work Items

### 1. Create Package Skeleton

Create `qa-planner/` with Codex and Claude plugin wrappers plus `skills/qa-planner/` portable skill.

Acceptance criteria:
- Folder structure matches target structure.
- `.codex-plugin/plugin.json` exists.
- `.claude-plugin/plugin.json` exists.
- Package is self-contained.
- `CHANGELOG.md` exists and records initial package version.
- No Codex-only dependency in core instructions.

### 2. Add `SKILL.md`

Create portable skill instructions.

Content must include:
- Purpose and triggers.
- Scope and out-of-scope rules.
- Workflow: intake, normalize, analyze, plan, test case design, coverage map, contract generation, human review, revision.
- `OK/NOK` review loop behavior.
- Template field requirements.
- Handoff behavior for `qa-executor`, `qa-automation`, `qa-reporter`.
- Quality gates.
- Platform fallback behavior.

Acceptance criteria:
- Skill can be read standalone by another agent.
- Instructions are specific enough to generate consistent output.
- No instruction tells planner to execute tests or write final automation scripts.

### 3. Add JSON Schemas

Create:
- `planning-state.schema.json`
- `handoff-contract.schema.json`
- `review-feedback.schema.json`

Schema rules:
- `planning-state.schema.json` must enforce metadata, source inputs, planning status, test cases, coverage map, handoff contracts, review history, and changelog.
- `handoff-contract.schema.json` must support target contracts for executor, automation, and reporter.
- `review-feedback.schema.json` must support `OK/NOK`, target level, target ref, action, expected change, and resolution.

Acceptance criteria:
- Schemas are valid JSON Schema draft 2020-12.
- Priority enum is exactly `P0 - Critical`, `P1 - High`, `P2 - Medium`, `P3 - Low`.
- Test case status enum is exactly `Untested`, `On Progress`, `Passed`, `Failed`, `Blocked`, `Retest`.
- Automation status enum is exactly `Manual Only`, `To Be Automate`, `Already Automate`.

### 4. Add Templates

Copy `Template Test Case.xlsx` into `qa-planner/templates/`.

Create Markdown templates:
- `test-plan.md`
- `test-case.md`
- `handoff-contract.json`

Acceptance criteria:
- Test case Markdown template preserves all Excel field names logically.
- Handoff template includes executor, automation, and reporter sections.
- Excel template remains unchanged except copy location.

### 5. Add Examples

Create examples:
- `input-requirement.md`: sample raw requirement.
- `planning-state.example.json`: realistic planner state.
- `output-test-cases.example.md`: human-readable test case table.

Acceptance criteria:
- Example uses approved priority/status values.
- Example demonstrates at least one positive, negative, and edge test case.
- Example includes one automation candidate and one manual-only case.
- Example includes review-ready handoff contract references.

### 6. Add README

Create `README.md` and `CHANGELOG.md` for humans installing, reviewing, and tracking package changes.

Content must include:
- What `qa-planner` does.
- Package contents.
- How to use with an agent platform.
- Expected inputs and outputs.
- Review loop explanation.
- Downstream agent handoff summary.

Acceptance criteria:
- README is short but complete.
- It does not assume one specific platform.
- `CHANGELOG.md` records initial version, compatibility targets, schema/template additions, and future migration notes.

### 7. Validate Package

Run local checks:
- Confirm file structure.
- Validate JSON syntax.
- Search for placeholder text.
- Confirm copied Excel template exists.
- Confirm references in README/SKILL match actual paths.

Acceptance criteria:
- No invalid JSON.
- No unresolved placeholder markers.
- All referenced files exist.

## Implementation Order

1. Create plugin and skill skeleton.
2. Copy Excel template.
3. Write schemas.
4. Write `SKILL.md`.
5. Write Markdown/JSON templates.
6. Write examples.
7. Write `README.md` and `CHANGELOG.md`.
8. Validate.
9. Summarize generated package and any limitations.

## Risks

- JSON schemas may become too strict for future platform-specific fields. Mitigation: allow controlled extension fields such as `tags`, `custom_fields`, and `references`.
- Test case template may evolve later. Mitigation: isolate field mapping in `SKILL.md` and schema definitions.
- Different platforms may not support Excel/Word/PDF rendering. Mitigation: Markdown + JSON fallback remains mandatory.

## Done Criteria

Implementation is done when:
- `qa-planner/` package exists.
- All target files exist.
- `.codex-plugin/plugin.json` exists and parses.
- `.claude-plugin/plugin.json` exists and parses.
- `CHANGELOG.md` exists with an initial entry.
- JSON files parse correctly.
- Package contains no unresolved placeholders.
- `Template Test Case.xlsx` is included under templates.
- Final summary lists created files and validation result.

