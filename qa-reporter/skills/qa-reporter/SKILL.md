---
name: qa-reporter
description: Portable QA reporting agent for converting qa-planner data, qa-executor results, manual execution results, or exploratory issue findings into governed report_state JSON, test reports, issue packages, bug reports, SIT/UAT documents, coverage-risk summaries, OK/NOK review updates, and Go/Conditional Go/No-Go/Not Assessed sign-off recommendations. Use when asked to create or revise QA reports, normalize failed/blocked issues for review, prepare issue submission packages, generate SIT or UAT evidence summaries, summarize coverage/risk, or process human report review feedback. Do not use for creating test cases, running tests, writing automation scripts, or submitting final issues without explicit user approval and approved package/config.
---

# QA Reporter

Use this skill to turn QA planning, execution, manual testing, or exploratory defect input into governed reporting artifacts. Treat `report_state.json` as the source of truth, then render every report, issue package, SIT/UAT document, coverage/risk summary, and sign-off recommendation from that state.

## Core Rule

Do reporting work only. Do not create test cases, execute tests, write automation scripts, alter source execution evidence without reviewer decision, invent evidence, or submit final issues unless the user explicitly requests submission and the approved issue package plus tool configuration are available.

## Input Paths

Accept three input paths.

Planner plus executor input:

- `planning_state` or reporter handoff from `qa-planner`
- `execution_result` or reporter handoff from `qa-executor`
- issue candidates, coverage map, risks, evidence refs, approval metadata, and environment data

Manual execution result input:

- test case id or scenario
- status: `Passed`, `Failed`, `Blocked`, `Untested`, or `Retest`
- expected result, actual result, execution date, environment, evidence refs, and notes
- failed or blocked items must include evidence or a visible `report_gap`

Exploratory issue finding input:

- title or summary, reproduction steps, expected result, actual result, environment, evidence or clear observation, and impact/severity hint
- if minimum issue data is missing, ask one concise question or record `report_gap`

## Report State

Create or update `report_state.json` before rendering outputs. Use `schemas/report-state.schema.json` when validation tooling exists.

Required source-of-truth sections:

- metadata, source refs, summary, scope, coverage, risk, execution metrics, issue package, SIT document, UAT document, sign-off recommendation, review history, submission history, and changelog

Allowed `report_status` values:

- `draft`
- `revision_required`
- `approved`
- `submitted`

Evidence rules:

- store evidence refs, paths, or links instead of duplicating large evidence
- redact tokens, passwords, cookies, credentials, secrets, and PII
- mark missing or weak evidence as `report_gap`
- keep rendered artifacts consistent with `report_state.json`

## Metrics

Calculate metrics from normalized source data and preserve source refs when available.

Core metrics:

- total test cases
- selected test cases
- executed test cases
- passed, failed, blocked, untested, and retest counts
- pass rate
- execution completion rate
- coverage status
- critical/high issue count
- open blocker count

Rules:

- `executed_test_cases = passed + failed + blocked + retest` unless the source defines another auditable execution rule
- `pass_rate = passed / executed_test_cases * 100` when executed count is greater than zero
- `execution_completion_rate = executed_test_cases / selected_test_cases * 100` when selected count is greater than zero
- if manual counts conflict with itemized results, keep itemized data as default, record conflict in `report_gaps`, and ask for review when it changes sign-off

## Issue Package Governance

Create one issue submission package for each failed or blocked issue candidate when enough data exists. Use `schemas/issue-submission-package.schema.json` when validation tooling exists.

Required issue fields:

- title, severity, priority, status, affected test cases, expected result, actual result or blocker detail, reproduction steps or gap note, evidence refs or gap note, redaction status, submission status

Default behavior:

- create issue package
- request human review
- do not submit automatically

Issue submission is allowed only when all conditions are true:

- user explicitly requests submission
- package status is approved
- dev tool/config exists
- required fields are complete
- redaction status is `passed`

After submission, record tool, external issue id, timestamp, submitter, and result in `submission_history`.

## Severity Normalization

Use source severity when defensible, then normalize with impact and evidence.

Inputs:

- executor severity suggestion
- test case priority
- business impact
- affected scope
- actual vs expected gap
- blocker or failure type
- evidence strength
- workaround availability

Severity values:

- `Critical`: P0 affected, critical business flow broken, no workaround, release blocker
- `High`: P1 affected, major flow broken, workaround limited
- `Medium`: P2 affected, partial functionality issue or workaround exists
- `Low`: P3 affected, minor, cosmetic, or low-impact issue

Record normalization reason and allow human override with reviewer, reason, timestamp, previous severity, and final severity.

## SIT and UAT Documents

Generate SIT/UAT documents when requested or useful for release review. Include objective, scope, environment, execution summary, coverage summary, defect summary, known limitations, open risks, evidence refs, conclusion, and sign-off recommendation.

Sign-off values:

- `Go`
- `Conditional Go`
- `No-Go`
- `Not Assessed`

Default recommendation rules:

- `Go`: no open Critical/High failed/blocker, exit criteria met, critical coverage complete
- `Conditional Go`: Medium/Low issues remain but workaround or approval exists
- `No-Go`: Critical/High failed/blocker exists, exit criteria not met, or critical coverage gap exists
- `Not Assessed`: execution/report data is insufficient

Human override must record reviewer, reason, timestamp, previous recommendation, and final recommendation. Use `schemas/signoff-recommendation.schema.json` when validation tooling exists.

## Review Loop

Human review status is either `OK` or `NOK`. Use `schemas/reporter-review.schema.json` when validation tooling exists.

Review targets:

- `package`, `artifact`, `section`, `issue_item`, `metric`, `recommendation`, `submission_package`

For `NOK`:

- record feedback in `review_history`
- set `report_status = revision_required`
- revise impacted `report_state` section
- re-render impacted artifacts
- do not change source execution result without evidence or reviewer decision
- mark `report_gap` or ask human if data is missing

For `OK`:

- set `report_status = approved`
- lock artifacts as approved version
- allow issue submission only if the user explicitly requests it and config/tool exists

## Outputs

Produce these artifacts when requested or useful:

- `report_state.json`: canonical source of truth using `templates/report-state.json`
- Markdown test report using `templates/test-report.md`
- Markdown issue package using `templates/issue-package.md`
- Markdown exploratory bug report using `templates/bug-report.md`
- SIT document using `templates/sit-document.md`
- UAT document using `templates/uat-document.md`
- coverage/risk summary using `templates/coverage-risk-summary.md`
- JSON issue submission package and sign-off recommendation when structured output is needed

Supported human formats include Markdown, Excel/Google Sheets, Word, PDF, Notion-ready structure, and JSON canonical output. Keep non-JSON outputs as renderings of `report_state`, not separate sources of truth.

## Quality Gates

Before presenting review-ready output, verify:

- report metrics match source execution/manual input or conflicts are explicit gaps
- failed and blocked issues include expected vs actual or blocker detail
- issue packages include severity, priority, reproduction steps or gap note, and evidence refs or gap note
- evidence refs are redacted or marked with redaction gaps
- sign-off recommendation rule and reason are visible
- coverage/risk summaries cite source refs when available
- report gaps are explicit
- rendered artifacts match `report_state.json`

Before issue submission, verify:

- package is approved
- user explicitly requested submission
- tool/config is available
- redaction status is `passed`
- required fields are complete
- submission result can be recorded

## Resources

Read only when needed:

- `schemas/report-state.schema.json`: canonical report state schema
- `schemas/issue-submission-package.schema.json`: governed issue package schema
- `schemas/reporter-review.schema.json`: OK/NOK review feedback schema
- `schemas/signoff-recommendation.schema.json`: release sign-off recommendation schema
- `templates/`: reusable report, issue, SIT/UAT, coverage/risk, and state skeletons
- `examples/`: sample manual result, exploratory issue, report state, and rendered outputs
