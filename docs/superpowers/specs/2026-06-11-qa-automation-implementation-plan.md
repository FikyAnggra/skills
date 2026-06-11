# QA-Automation Implementation Plan

Date: 2026-06-11
Spec: `docs/superpowers/specs/2026-06-11-qa-automation-design.md`
Status: Draft for implementation approval

## Goal

Build a portable `qa-automation` agent skill package that creates or updates automation scripts through a governed change-set workflow.

The package must follow the same portable plugin/skill pattern used by the other QA agents.

## Target Structure

```text
qa-automation/
  .codex-plugin/
    plugin.json
  .claude-plugin/
    plugin.json
  skills/
    qa-automation/
      SKILL.md
      schemas/
        automation-change-set.schema.json
        automation-review.schema.json
        validation-result.schema.json
        automation-status-update.schema.json
      templates/
        automation-change-set.json
        automation-review.md
        validation-report.md
        patch-summary.md
        promotion-checklist.md
      references/
        playwright-typescript.md
        locator-policy.md
        branch-policy.md
      examples/
        planner-automation-contract.json
        executor-automation-signal.json
        automation-change-set.example.json
        output-automation-review.example.md
      agents/
        openai.yaml
  README.md
  CHANGELOG.md
```

## Work Items

### 1. Create Plugin and Skill Skeleton

Create `qa-automation/` plugin wrapper and `skills/qa-automation/` portable skill.

Acceptance criteria:
- `.codex-plugin/plugin.json` exists.
- `.claude-plugin/plugin.json` exists.
- `skills/qa-automation/SKILL.md` exists.
- Package is self-contained.
- `CHANGELOG.md` exists and records initial package version.
- Core skill instructions stay platform-neutral.

### 2. Write `SKILL.md`

Create portable automation instructions.

Content must include:
- purpose and triggers
- input paths: planner contract, executor signal, manual request
- `automation_change_set.json` source of truth
- repo/framework detection policy
- Playwright TypeScript fallback
- locator policy summary
- script generation rules
- validation tiers
- script review gate
- promotion review gate
- branch policy
- output and downstream signals
- safety and quality gates

Acceptance criteria:
- Skill can be read standalone by another agent.
- Skill does not tell automation agent to run manual QA as executor, create test cases from requirements, write final reports, or submit issues.
- Skill does not push/merge remote without explicit user request/approval.

### 3. Add JSON Schemas

Create:
- `automation-change-set.schema.json`
- `automation-review.schema.json`
- `validation-result.schema.json`
- `automation-status-update.schema.json`

Schema rules:
- `automation-change-set.schema.json` supports repo context, framework, branch policy, automation scope, mappings, planned/applied changes, validation results, review gates, promotion status, gaps, and changelog.
- `automation-review.schema.json` supports script review and promotion review OK/NOK feedback.
- `validation-result.schema.json` supports `lint`, `typecheck`, `format_check`, `config_validation`, `targeted_run`, `full_run`.
- `automation-status-update.schema.json` supports previous/new automation status and script validation outcome.

Acceptance criteria:
- All schema files are valid JSON.
- Enums match approved design exactly.
- Extension fields are controlled through `custom_fields` where needed.

### 4. Add Templates

Create:
- `automation-change-set.json`
- `automation-review.md`
- `validation-report.md`
- `patch-summary.md`
- `promotion-checklist.md`

Acceptance criteria:
- Templates render from `automation_change_set`.
- Automation review includes TC mapping, planned/applied changes, assertions, locators, gaps, and validation results.
- Validation report includes command, status, timestamps, summary, and evidence refs.
- Patch summary includes files created/updated/deleted and rationale.
- Promotion checklist includes branch policy, review status, validation status, gaps, and remote action approval state.

### 5. Add References

Create:
- `playwright-typescript.md`
- `locator-policy.md`
- `branch-policy.md`

Acceptance criteria:
- Playwright reference includes default project structure, POM guidance, fixture guidance, config notes, and command examples.
- Locator policy includes priority order and anti-fragile selector rules.
- Branch policy includes follow-repo-first rule, fallback `develop -> master`, PR/feature branch support, and explicit approval for remote actions.

### 6. Add Examples

Create:
- `planner-automation-contract.json`
- `executor-automation-signal.json`
- `automation-change-set.example.json`
- `output-automation-review.example.md`

Acceptance criteria:
- Planner contract example includes create/update/manual-only cases.
- Executor signal example includes selector gap, flaky technical failure, and update candidate.
- Change set example maps TC ID to script path and validation result.
- Automation review example shows script review status, promotion readiness, and gaps.

### 7. Add README

Create `README.md` and `CHANGELOG.md` for humans installing, reviewing, and tracking package changes.

Content must include:
- what `qa-automation` does
- package contents
- input paths
- Playwright TypeScript fallback
- locator policy summary
- validation policy
- review gates
- promotion policy
- downstream outputs

Acceptance criteria:
- README is short but complete.
- README does not assume one specific agent platform.
- `CHANGELOG.md` records initial version, compatibility targets, schema/template additions, and future migration notes.

### 8. Generate Skill UI Metadata

Create `agents/openai.yaml` for Codex-compatible display metadata.

Acceptance criteria:
- File exists under `skills/qa-automation/agents/openai.yaml`.
- Metadata matches `SKILL.md` behavior.

### 9. Validate Package

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
6. Write references.
7. Write examples.
8. Write README.
9. Generate `agents/openai.yaml`.
10. Validate.
11. Summarize generated package and limitations.

## Risks

- Existing repos may use unknown framework patterns. Mitigation: inspect first, follow existing style, and mark framework gaps.
- Selector data may be missing. Mitigation: record selector gaps and propose testability improvements.
- Generated tests may need environment/test data. Mitigation: record env/data gaps and avoid false validation claims.
- Validation commands may be unavailable or costly. Mitigation: run light validation when available, targeted run when safe, full run only explicit/policy.
- Remote git actions are risky. Mitigation: require explicit user approval for push/merge.

## Done Criteria

Implementation is done when:
- `qa-automation/` package exists.
- All target files exist.
- `.codex-plugin/plugin.json` exists and parses.
- `.claude-plugin/plugin.json` exists and parses.
- `CHANGELOG.md` exists with an initial entry.
- JSON files parse correctly.
- Package contains no unresolved placeholder markers.
- `SKILL.md` frontmatter is valid.
- README explains input paths, fallback framework, locator policy, validation, review, and promotion.
- Final summary lists created files and validation result.
