# QA-Executor Implementation Plan

Date: 2026-06-11
Spec: `docs/superpowers/specs/2026-06-11-qa-executor-design.md`
Status: Draft for implementation approval

## Goal

Build a portable `qa-executor` agent skill package that can execute or guide execution of QA test cases, capture evidence, produce `execution_result.json`, and hand off structured results to reporter and automation agents.

The package must follow the same portable plugin/skill pattern used by `qa-planner`.

## Target Structure

```text
qa-executor/
  .codex-plugin/
    plugin.json
  .claude-plugin/
    plugin.json
  skills/
    qa-executor/
      SKILL.md
      schemas/
        execution-package.schema.json
        execution-result.schema.json
        execution-review.schema.json
        issue-candidate.schema.json
      templates/
        execution-report.md
        execution-result.json
        issue-candidate.json
        qa-reporter-handoff.json
        qa-automation-signal.json
      examples/
        manual-test-cases.md
        execution-package.example.json
        execution-result.example.json
        output-execution-report.example.md
      agents/
        openai.yaml
  README.md
  CHANGELOG.md
```

## Work Items

### 1. Create Plugin and Skill Skeleton

Create `qa-executor/` plugin wrapper and `skills/qa-executor/` portable skill.

Acceptance criteria:
- `.codex-plugin/plugin.json` exists.
- `.claude-plugin/plugin.json` exists.
- `skills/qa-executor/SKILL.md` exists.
- Package is self-contained.
- `CHANGELOG.md` exists and records initial package version.
- Core skill instructions stay platform-neutral.

### 2. Write `SKILL.md`

Create portable executor instructions.

Content must include:
- purpose and triggers
- planner-driven and manual input paths
- execution package normalization
- run policy and execution scope resolution
- API/UI/DB/manual execution modes
- status rules
- blocked rules
- retry policy
- evidence and redaction rules
- issue candidate behavior
- human OK/NOK review loop
- reporter handoff and automation signal behavior
- quality gates

Acceptance criteria:
- Skill can be read standalone by another agent.
- Skill does not tell executor to create test plans, submit final issues, or write final automation scripts.
- Skill supports platforms without API/UI/DB tools by producing manual execution instructions and structured result templates.

### 3. Add JSON Schemas

Create:
- `execution-package.schema.json`
- `execution-result.schema.json`
- `execution-review.schema.json`
- `issue-candidate.schema.json`

Schema rules:
- Support both `qa-planner` handoff input and manual test case input.
- Enforce run policy modes: `initial_run`, `full_regression`, `failed_retest`, `smoke`, `targeted`, `automation_validation`.
- Enforce execution modes: `api`, `ui`, `db`, `hybrid`, `manual_instruction`.
- Enforce test case statuses: `Untested`, `On Progress`, `Passed`, `Failed`, `Blocked`, `Retest`.
- Enforce blocker types: `test_dependency`, `environment`, `data`, `access`, `credential`, `endpoint`, `db`, `fixture`, `setup`.
- Enforce issue candidate fields for `Failed` and `Blocked` outputs.

Acceptance criteria:
- All schema files are valid JSON.
- Enums match approved design exactly.
- Extension fields are controlled through `custom_fields` where needed.

### 4. Add Templates

Create:
- `execution-report.md`
- `execution-result.json`
- `issue-candidate.json`
- `qa-reporter-handoff.json`
- `qa-automation-signal.json`

Acceptance criteria:
- Report template includes summary counts, per-test status, evidence, blockers, retries, and issue candidates.
- Result template matches `execution-result.schema.json` shape.
- Issue candidate template is reporter-ready but not final submission.
- Reporter handoff and automation signal are compact and reference execution result.

### 5. Add Examples

Create:
- `manual-test-cases.md`
- `execution-package.example.json`
- `execution-result.example.json`
- `output-execution-report.example.md`

Acceptance criteria:
- Example includes one `Passed`, one `Failed`, and one `Blocked` result.
- Example shows manual input normalization.
- Example includes API/UI/DB evidence examples where relevant.
- Example includes retry evidence for technical flake.
- Example includes issue candidate for `Failed` or `Blocked`.
- Example includes reporter handoff and automation signal references.

### 6. Add README

Create `README.md` and `CHANGELOG.md` for humans installing, reviewing, and tracking package changes.

Content must include:
- what `qa-executor` does
- package contents
- how to use with planner contract or manual test cases
- expected outputs
- status model
- review loop
- downstream handoff summary

Acceptance criteria:
- README is short but complete.
- README does not assume one specific agent platform.
- `CHANGELOG.md` records initial version, compatibility targets, schema/template additions, and future migration notes.

### 7. Generate Skill UI Metadata

Create `agents/openai.yaml` for Codex-compatible display metadata.

Acceptance criteria:
- File exists under `skills/qa-executor/agents/openai.yaml`.
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

- Execution schemas may become too strict for platform-specific evidence. Mitigation: allow controlled `custom_fields` and evidence references.
- Manual test case formats may vary. Mitigation: normalize minimum fields and ask when expected result is missing.
- Tools may not support live API/UI/DB execution. Mitigation: require `manual_instruction` fallback.
- Evidence may contain secrets. Mitigation: skill must redact token, password, cookie, credential, secret, and PII.

## Done Criteria

Implementation is done when:
- `qa-executor/` package exists.
- All target files exist.
- `.codex-plugin/plugin.json` exists and parses.
- `.claude-plugin/plugin.json` exists and parses.
- `CHANGELOG.md` exists with an initial entry.
- JSON files parse correctly.
- Package contains no unresolved placeholder markers.
- `SKILL.md` frontmatter is valid.
- README explains planner-driven and manual execution paths.
- Final summary lists created files and validation result.
