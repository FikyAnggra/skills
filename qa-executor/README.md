# QA Executor

`qa-executor` is a portable QA execution skill package. It runs approved test cases when tools and access are safe, produces manual execution instructions when they are not, captures redacted evidence, and writes `execution_result.json` for downstream QA reporting and automation agents.

## Contents

- `.codex-plugin/plugin.json`: Codex plugin wrapper.
- `.claude-plugin/plugin.json`: Claude-compatible wrapper.
- `skills/qa-executor/SKILL.md`: canonical platform-neutral executor instructions.
- `skills/qa-executor/schemas/`: JSON schemas for execution packages, results, reviews, and issue candidates.
- `skills/qa-executor/templates/`: reusable result, report, issue, reporter handoff, and automation signal templates.
- `skills/qa-executor/examples/`: manual input and sample execution outputs.

## Usage

Use planner-driven input when `qa-planner` has produced a `qa_executor` handoff contract with approved cases, environment constraints, test data refs, and run policy.

Use manual input when a human provides test cases from Excel, Google Sheets, Markdown, Word/PDF tables, Notion, JSON, or plain text. Minimum manual fields are `TC ID`, `Scenario`, `Test Step`, and `Expected Result`.

Expected outputs are:
- `execution_result.json` as source of truth.
- Markdown execution report.
- spreadsheet-compatible status/result updates.
- issue candidates for failed and blocked cases.
- `qa_reporter_handoff` after human OK review.
- `qa_automation_signal` when automation gaps or update candidates exist.

## Status Model

Allowed test case statuses are `Untested`, `On Progress`, `Passed`, `Failed`, `Blocked`, and `Retest`. Failed cases need expected vs actual results. Blocked cases need blocker type and reason. Dependency blocks must identify `blocked_by` test case ids.

## Review Loop

Execution output goes through human `OK` or `NOK` review. `OK` accepts the package and enables reporter and automation handoffs. `NOK` records feedback, patches documentation or evidence, and reruns only when new evidence is required and safe.

## Downstream Handoff

`qa-reporter` consumes execution summary, evidence refs, issue candidates, coverage/risk refs, and approval metadata. `qa-automation` consumes flaky technical failures, selector/API/data gaps, and automation update candidates. This package does not submit final issues or write final automation scripts.
