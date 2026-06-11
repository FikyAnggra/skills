# QA-Reporter Portable Agent Skill Design

Date: 2026-06-11
Status: Approved design draft for user review
Source artifacts:
- `QA Agent & Workflow.pdf`
- `docs/superpowers/specs/2026-06-10-qa-planner-design.md`
- `docs/superpowers/specs/2026-06-11-qa-executor-design.md`

## 1. Purpose

`qa-reporter` is a portable governance reporting agent skill/package. It converts planning data, execution results, manual testing results, or exploratory issue findings into reporting packages for human review, audit, issue submission, SIT/UAT documentation, and release sign-off decisions.

The reporter creates a canonical `report_state.json` and renders multi-format human artifacts from that state.

## 2. Scope

In scope:
- Generate test report.
- Generate issue package.
- Generate bug report from exploratory findings.
- Generate SIT document.
- Generate UAT document.
- Generate coverage summary.
- Generate risk summary.
- Generate release sign-off summary.
- Normalize issue candidates into issue submission packages.
- Normalize manual execution results into report state.
- Normalize exploratory issue findings into report state.
- Produce `Go`, `Conditional Go`, `No-Go`, or `Not Assessed` recommendation.
- Support multi-level `OK/NOK` review loop.
- Submit approved issue package to dev tool only when user explicitly asks and tool/config exists.

Out of scope:
- Creating test cases.
- Running tests.
- Creating or updating automation scripts.
- Changing original execution result without evidence or reviewer decision.
- Submitting issue final without explicit user request and approved package/config.
- Inventing evidence, execution result, environment, or reproduction steps.

## 3. Input Paths

`qa-reporter` supports three input paths.

### 3.1 Planner + Executor Input

Inputs:
- `planning_state`
- `execution_result`
- issue candidates
- coverage map
- risk analysis
- evidence references
- planning approval metadata
- execution approval metadata

Use this path for standard workflow from `qa-planner` and `qa-executor`.

### 3.2 Manual Execution Result Input

Inputs from human manual testing:
- test case id or scenario
- status: `Passed`, `Failed`, `Blocked`, `Untested`, or `Retest`
- expected result
- actual result
- execution date
- environment
- evidence or notes for `Failed` and `Blocked`

Outputs can include:
- test report
- SIT/UAT document
- issue package for `Failed` or `Blocked`
- coverage/risk summary if mapping exists

### 3.3 Exploratory Issue Finding Input

Inputs from exploratory testing:
- title or summary
- reproduction steps
- expected result
- actual result
- environment
- evidence or clear observation
- impact/severity hint

Outputs can include:
- bug report
- issue report
- issue submission package
- evidence gap checklist

If minimum information is missing, ask a concise question or mark `report_gap`.

## 4. Report State as Source of Truth

`report_state.json` is the canonical reporter state. All human artifacts are rendered from it.

Principles:
- Do not duplicate large evidence.
- Store evidence references, paths, or links.
- Store normalized summary, metrics, issues, recommendation, and review history.
- Apply human feedback to `report_state`, then re-render impacted outputs.

Top-level structure:

```json
{
  "report_id": "RPT-20260611-001",
  "source_agent": "qa-reporter",
  "package_id": "QA-PLAN-AUTH-LOGIN-v0.2",
  "planning_state_ref": "planning_state.json",
  "execution_result_ref": "execution_result.json",
  "report_status": "draft",
  "metadata": {},
  "summary": {},
  "scope": {},
  "coverage": {},
  "risk": {},
  "execution_metrics": {},
  "issue_package": {},
  "sit_document": {},
  "uat_document": {},
  "signoff_recommendation": {},
  "review_history": [],
  "submission_history": [],
  "changelog": []
}
```

Report status:
- `draft`
- `revision_required`
- `approved`
- `submitted`

Core metrics:
- total test cases
- selected test cases
- executed test cases
- passed
- failed
- blocked
- untested
- retest
- pass rate
- execution completion rate
- coverage status
- critical/high issue count
- open blocker count

Evidence handling:
- `report_state` stores `evidence_refs`.
- Sensitive values must remain redacted.
- Missing evidence becomes `report_gap`.

## 5. Issue Governance and Submission

`qa-reporter` receives issue candidates from `qa-executor` or exploratory/manual input, then normalizes them into `issue_submission_package`.

Default behavior:
- create issue package
- request human review
- do not submit automatically

Issue submission is allowed only if:
- user explicitly requests submit
- issue package is approved
- dev tool/config exists
- required fields are complete
- redaction status is passed

Severity normalization inputs:
- executor severity suggestion when available
- test case priority
- business impact
- affected scope
- actual vs expected gap
- blocker or failure type
- evidence strength
- workaround availability

Default severity mapping:
- `Critical`: P0 affected, critical business flow broken, no workaround, release blocker.
- `High`: P1 affected, major flow broken, workaround limited.
- `Medium`: P2 affected, partial functionality issue or workaround exists.
- `Low`: P3 affected, minor/cosmetic/low-impact issue.

Issue submission package example:

```json
{
  "issue_id": "ISSUE-PKG-TC0002",
  "source_issue_candidate_id": "ISSUE-CAND-TC0002",
  "title": "Login accepts invalid password",
  "severity": "High",
  "priority": "P1",
  "status": "ready_for_review",
  "affected_test_cases": ["TC0002"],
  "expected_result": "Generic invalid credential message is shown.",
  "actual_result": "User landed on dashboard.",
  "reproduction_steps": [],
  "evidence_refs": [],
  "suggested_owner": "dev",
  "redaction_status": "passed",
  "submission_status": "not_submitted"
}
```

Submission result example:

```json
{
  "submitted": true,
  "tool": "jira|linear|github|other",
  "external_issue_id": "PROJ-123",
  "submitted_at": "2026-06-11T10:30:00+07:00",
  "submitted_by": "qa-reporter"
}
```

## 6. Report Types

`qa-reporter` can generate these report types:
- `test_report`
- `issue_report`
- `bug_report`
- `issue_submission_package`
- `sit_document`
- `uat_document`
- `coverage_summary`
- `risk_summary`
- `release_signoff_summary`

Default rendered outputs:
- `test-report.md`
- `issue-package.md`
- `sit-document.md`
- `uat-document.md`
- `coverage-risk-summary.md`
- `report_state.json`

Supported human formats:
- Markdown
- Excel/Google Sheets
- Word
- PDF
- Notion-ready structure
- JSON canonical output

## 7. SIT and UAT Documents

SIT/UAT documents include:
- objective
- scope
- environment
- execution summary
- coverage summary
- defect/issue summary
- known limitations
- open risks
- evidence references
- conclusion
- sign-off recommendation

Sign-off recommendation values:
- `Go`
- `Conditional Go`
- `No-Go`
- `Not Assessed`

Default recommendation rules:
- `Go`: no open Critical/High failed/blocker, exit criteria met, critical coverage complete.
- `Conditional Go`: Medium/Low issues remain but workaround or approval exists.
- `No-Go`: Critical/High failed/blocker exists, exit criteria not met, or critical coverage gap exists.
- `Not Assessed`: execution/report data is insufficient.

Human override is allowed, but must record:
- reviewer
- reason
- timestamp
- previous recommendation
- final recommendation

## 8. Review Loop

Reporter review uses multi-level `OK/NOK`.

Review targets:
- `package`
- `artifact`
- `section`
- `issue_item`
- `metric`
- `recommendation`
- `submission_package`

Example review feedback:

```json
{
  "review_status": "NOK",
  "target_level": "metric",
  "target_ref": "execution_metrics.pass_rate",
  "issue": "Pass rate does not match manual testing result.",
  "action": "update",
  "expected_change": "Recalculate based on 8 passed from 10 executed."
}
```

If `NOK`:
- record feedback in `review_history`
- set `report_status = revision_required`
- revise impacted `report_state` section
- re-render impacted artifact
- do not change original execution result without evidence or reviewer decision
- mark `report_gap` or ask human if data is missing

If `OK`:
- set `report_status = approved`
- lock artifacts as approved version
- allow issue submission only if user explicitly requests it and config/tool exists

## 9. Quality Gates

Before review-ready output:
- report metrics match source execution/manual input
- failed/blocked issues have expected vs actual
- issue packages have severity, priority, reproduction steps or gap note, evidence refs or gap note
- evidence remains redacted
- sign-off recommendation rule and reason are visible
- coverage/risk summaries cite source refs when available
- report gaps are explicit
- rendered artifacts match `report_state.json`

Before issue submission:
- issue package approved
- user explicitly requested submission
- tool/config available
- redaction status passed
- required fields complete
- submission result is recorded

## 10. Portable Package Structure

Target package:

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

Core behavior must be platform-neutral. Codex and Claude plugin wrappers are runtime packaging layers, not required for agent behavior. `CHANGELOG.md` records versioned changes, compatibility updates, schema/template changes, and migration notes.

## 11. Implementation Notes

Implementation should produce:
- portable `qa-reporter` skill folder
- Codex plugin wrapper matching `qa-planner` package pattern
- Claude plugin wrapper matching the same package behavior
- schemas for report state, issue submission package, reporter review, and sign-off recommendation
- templates for reports, issue packages, SIT/UAT documents, coverage/risk summary, and report state
- examples for planner/executor input, manual execution, and exploratory issue input
- `README.md` and `CHANGELOG.md`
- validation checks for JSON, frontmatter, placeholder scan, and required files

No platform-specific issue tracker integration is mandatory. If a platform lacks issue submission tooling, `qa-reporter` must still generate an approved issue submission package.
