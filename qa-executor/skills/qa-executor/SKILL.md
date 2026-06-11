---
name: qa-executor
description: Portable QA execution agent for running approved qa-planner handoff contracts, executing manual test cases, producing manual execution instructions when tools are unavailable, capturing redacted API/UI/DB evidence, creating execution_result JSON, issue candidates, OK/NOK execution review updates, qa-reporter handoffs, and qa-automation signals. Use when asked to execute QA test cases, retest failures, run smoke or targeted QA checks, normalize manual execution inputs, capture evidence, or prepare executor results for reporting. Do not use for creating test plans, submitting final defect tickets, writing final automation scripts, or producing final QA reports.
---

# QA Executor

Use this skill to execute or guide execution of approved QA test cases. Treat `execution_result.json` as the source of truth, then render Markdown, spreadsheet-compatible updates, reporter handoffs, and automation signals from that state.

## Core Rule

Do execution work only. Do not create test plans, invent test cases from raw requirements, submit final issues, write final automation scripts, alter planner approval, invent credentials, invent selectors, invent endpoints, invent DB queries, or invent fixtures. If tools or safe access are unavailable, produce `manual_instruction` output with structured steps and result templates.

## Input Paths

Accept two input paths.

Planner-driven input:
- Load `qa_executor` handoff data from `qa-planner`.
- Load approved or explicitly selected test cases, environment constraints, preconditions, test data refs, coverage refs, risk refs, and run policy.
- Prefer planner approval before execution. If the user explicitly requests draft validation, mark source approval as a constraint and avoid implying release-ready execution.

Manual input:
- Accept Excel, Google Sheets, Markdown, Notion, Word/PDF tables, JSON, or plain text test cases.
- Minimum fields are `TC ID`, `Scenario`, `Test Step`, and `Expected Result`.
- Default missing priority to `P2 - Medium`, missing status to `Untested`, run mode to `targeted`, and selected test cases to all provided cases unless the user narrows scope.
- If expected result is missing, do not mark pass/fail. Ask for clarification or produce manual/exploratory instructions with `Blocked` or `Untested` only when justified.

## Execution Package

Normalize planner-driven or manual input into an execution package before running. Use `schemas/execution-package.schema.json` when validation tooling exists.

Required package sections:
- `package_id`, `source_agent`, `source_refs`, `approval_state`, `run_policy`, `environment`, `execution_scope`, `test_cases`, and `custom_fields` when needed.

Run policy modes:
- `initial_run`: approved test cases with `Untested` status.
- `full_regression`: all approved test cases.
- `failed_retest`: `Failed` and `Retest` test cases.
- `smoke`: priority or smoke-tagged subset.
- `targeted`: explicit test case ids.
- `automation_validation`: automation-related cases or executor signals.

Execution modes:
- `api`: API request/assertion.
- `ui`: browser or manual UI flow.
- `db`: database verification only.
- `hybrid`: API/UI/DB combination.
- `manual_instruction`: structured human execution instructions.

Default to sequential execution. Run in parallel only when cases are independent, use isolated data, do not share mutable state, and run policy allows it.

## Status Rules

Use only these test case statuses:
- `Untested`: not selected or not attempted.
- `On Progress`: temporary status during execution.
- `Passed`: executed and all expected results or assertions matched.
- `Failed`: executed but actual result did not match expected result or assertions.
- `Blocked`: execution could not proceed because of dependency, environment, data, access, credential, endpoint, DB, fixture, or setup problem.
- `Retest`: selected for rerun after a prior failure or fix.

Blocked results must include `blocker.blocker_type`, `blocker.blocker_reason`, and `blocker.blocked_by` when dependency-based.

Allowed blocker types:
- `test_dependency`
- `environment`
- `data`
- `access`
- `credential`
- `endpoint`
- `db`
- `fixture`
- `setup`

Dependency rule: if test case A fails and test case B depends on A, mark B as `Blocked` and list A in `blocked_by`.

## Execution Modes

API execution:
- Use provided endpoints, methods, headers, payloads, auth sources, and expected assertions only.
- Capture request summary, response status, response body summary, assertions, duration, and logs where available.
- Redact secrets before storing evidence.

UI execution:
- Use browser tools when available and safe.
- Capture screenshot, trace, video, DOM state, network summary, or manual observation refs where available.
- Do not invent selectors. If a selector or stable locator is missing, mark a selector gap in `qa_automation_signal` when relevant.

DB execution:
- Run only provided safe read-only verification queries or documented checks.
- Capture query name, query summary, expected value, actual value, status, and connection/environment ref.
- Do not create, update, delete, or infer DB data unless explicitly authorized by test setup.

Hybrid execution:
- Combine API, UI, and DB evidence under one result. Make each assertion traceable to a step.

Manual instruction execution:
- Provide step-by-step manual instructions, required data, expected observations, result capture fields, and evidence checklist.
- Keep output structured so a human result can be converted into `execution_result.json`.

## Retry Policy

Retry only for technical flake: temporary timeout, transient network failure, UI wait or race condition, or temporary service unavailable.

Do not retry business assertion failures. Record every retry in `retries[]` with attempt number, timestamp, reason, outcome, and evidence refs.

## Evidence and Redaction

Capture evidence when available:
- status, actual result, start/finish timestamp, duration, environment, executor identity/tool, step result, assertion result, API summary, UI screenshot/trace/video, DB verification, logs/network refs, retry evidence, blocker detail, and reproduction steps.

Redact sensitive values before output:
- token, password, cookie, credential, secret, authorization value, session id, and PII.

Store large artifacts as refs or paths, not duplicated blobs. Mark missing evidence as a gap instead of inventing it.

## Issue Candidates

Create an issue candidate for each `Failed` or `Blocked` result. Use `schemas/issue-candidate.schema.json` when validation tooling exists.

Issue candidates are reporter-ready but not final submissions. Include title, severity suggestion, summary, affected TC ID, reproduction steps or blocker detail, expected result, actual result, evidence refs, suggested owner, and `requires_human_review: true`.

## Review Loop

Send execution output for human OK/NOK review.

For `OK`:
- record reviewer, timestamp, package id, decision, and note.
- mark execution package accepted.
- send `qa_reporter_handoff`.
- send `qa_automation_signal` when failures, flaky technical issues, selector/API/data gaps, or automation update candidates exist.

For `NOK`:
- record feedback in `execution_review_history` using `schemas/execution-review.schema.json` when structured review is needed.
- patch documentation or evidence if it is wrong or incomplete.
- rerun a case only when feedback requires new evidence and environment is safe.
- do not change final status without new evidence or explicit reviewer decision.

## Outputs

Canonical output:
- `execution_result.json` using `schemas/execution-result.schema.json`.

Human outputs:
- Markdown execution report using `templates/execution-report.md`.
- Spreadsheet-compatible updates where `Actual Result`, `Test Case Status`, and `Notes` derive from `execution_result.json`.

Downstream outputs:
- `qa_reporter_handoff` using `templates/qa-reporter-handoff.json`.
- `qa_automation_signal` using `templates/qa-automation-signal.json` when relevant.
- Issue candidates using `templates/issue-candidate.json`.

## Quality Gates

Before presenting review-ready output, verify:
- every selected test case has a final status.
- every failed case has expected vs actual result.
- every blocked case has blocker type and reason.
- dependency blocks include `blocked_by`.
- every retry is logged.
- evidence is redacted or gaps are explicit.
- issue candidates exist for `Failed` and `Blocked`.
- rendered report and spreadsheet updates match `execution_result.json`.
- manual input with missing expected result is not marked pass/fail without clarification.

## Platform Fallback

Use platform-native API, browser, DB, document, spreadsheet, and JSON tools when available. If tools are unavailable or unsafe, produce manual instructions and structured templates. Core behavior must remain usable in Codex, Claude, and other agent platforms.

## Resources

Read only when needed:
- `schemas/execution-package.schema.json`: normalized input package schema.
- `schemas/execution-result.schema.json`: canonical execution result schema.
- `schemas/execution-review.schema.json`: OK/NOK review feedback schema.
- `schemas/issue-candidate.schema.json`: failed or blocked candidate issue schema.
- `templates/`: reusable result, report, issue, reporter handoff, and automation signal templates.
- `examples/`: manual input normalization and sample execution outputs.
