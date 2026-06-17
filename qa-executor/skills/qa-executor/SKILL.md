---
name: qa-executor
description: Portable QA execution agent for approved qa-planner handoffs, manual test cases, Plane.so work items, and Notion pages/databases. Executes or guides API/UI/DB/manual QA, captures redacted evidence, writes execution_result JSON, syncs managed Plane/Notion execution output, creates issue candidates, handles OK/NOK review, and produces qa-reporter handoffs and qa-automation signals. Use for QA execution, retest, smoke/targeted runs, evidence capture, Plane/Notion execution sync, and execution result prep. Do not use for test planning, final defect submission, final automation scripts, or final QA reports.
---

# QA Executor

Use this skill to execute or guide execution of approved QA test cases. Treat `execution_result.json` as the source of truth, then render Markdown, spreadsheet-compatible updates, reporter handoffs, and automation signals from that state.

## Core Rule

Do execution work only. Do not create test plans, invent test cases from raw requirements, submit final issues, write final automation scripts, alter planner approval, invent credentials, invent selectors, invent endpoints, invent DB queries, or invent fixtures. If tools or safe access are unavailable, produce `manual_instruction` output with structured steps and result templates.

Plane and Notion are synced workflow surfaces, not the source of truth. Keep `execution_result.json` canonical and render Plane or Notion comments, pages, database rows, status moves, worklogs, links, and wiki pages from that state.

## Input Paths

Accept four input paths.

Planner-driven input:
- Load `qa_executor` handoff data from `qa-planner`.
- Load approved or explicitly selected test cases, environment constraints, preconditions, test data refs, coverage refs, risk refs, and run policy.
- Prefer planner approval before execution. If the user explicitly requests draft validation, mark source approval as a constraint and avoid implying release-ready execution.

Manual input:
- Accept Excel, Google Sheets, Markdown, Notion, Word/PDF tables, JSON, or plain text test cases.
- Minimum fields are `TC ID`, `Scenario`, `Test Step`, and `Expected Result`.
- Default missing priority to `P2 - Medium`, missing status to `Untested`, run mode to `targeted`, and selected test cases to all provided cases unless the user narrows scope.
- If expected result is missing, do not mark pass/fail. Ask for clarification or produce manual/exploratory instructions with `Blocked` or `Untested` only when justified.

Plane input:
- Accept Plane readable ids such as `ENG-42`, Plane URLs, UUIDs, or MCP payloads.
- Treat Plane work item/card descriptions, comments, linked refs, wiki/page refs, attachments, cycle/module context, relations, and worklogs as source context when tools allow.
- Read project states before status movement. Do not assume state names exist.
- Extract executable test cases only when Plane content contains a planner executor handoff, structured execution package, or minimum manual fields: `TC ID`, `Scenario`, `Test Step`, and `Expected Result`.
- If a Plane work item contains only requirements, acceptance criteria, or feature text, record `source_not_executable` and ask whether to route it to `qa-planner`. Do not invent test cases.
- Parse Plane comments or user text commands such as `run smoke`, `run TC0001, TC0004`, `retest failed`, `publish wiki`, and `sync reporter handoff` as requested policy signals. These commands do not bypass Plane write preflight.

Notion input:
- Accept Notion page URLs, database URLs, data source ids, page ids, or MCP payloads.
- Treat Notion page content, database rows, data source records, comments/discussions, linked pages, attached refs, and connected-source search results as source context when tools allow.
- Extract executable test cases only when Notion content contains a planner executor handoff, structured execution package, or minimum manual fields: `TC ID`, `Scenario`, `Test Step`, and `Expected Result`.
- Map Notion database columns to execution fields when names match or are obvious: `TC ID`, `Scenario`, `Test Step`, `Expected Result`, `Priority`, `Test Case Status`, `Actual Result`, and `Notes`.
- If a Notion page/database contains only requirements, acceptance criteria, or feature text, record `source_not_executable` and ask whether to route it to `qa-planner`. Do not invent test cases.
- Parse Notion comments or user text commands such as `run smoke`, `run TC0001, TC0004`, `retest failed`, `publish execution page`, `update test result database`, `sync reporter handoff`, `mark review OK`, and `NOK: <feedback>` as requested policy signals. These commands do not bypass Notion write preflight.

## Execution Package

Normalize planner-driven or manual input into an execution package before running. Use `schemas/execution-package.schema.json` when validation tooling exists.

Required package sections:
- `package_id`, `source_agent`, `source_refs`, `approval_state`, `run_policy`, `environment`, `execution_scope`, `test_cases`, and `custom_fields` when needed.

Plane package sections when Plane path is active:
- `plane_context`: workspace/project/work item ids, readable id, URL, state, assignees, labels, cycle/module refs, relations, comments, links, and attachment refs.
- `plane_write_policy`: comment, status, worklog, link, wiki, follow-up item, and confirmation settings.
- `plane_status_mapping`: target states for start, all passed, any failed, blocked, partial, and cancelled outcomes.

Notion package sections when Notion path is active:
- `notion_context`: source page/database/data source ids, URLs, title, source rows/pages, comments, linked refs, output target refs, and read gaps.
- `notion_write_policy`: source database update, execution page sync, execution database sync, comment sync, issue candidate sync, and confirmation settings.

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

## Plane Preflight

Before any Plane write, resolve write policy and status mapping.

Required status mapping keys:
- `on_start`
- `on_all_passed`
- `on_any_failed`
- `on_blocked`
- `on_partial`
- `on_cancelled`

Ask the user for this mapping when it is absent. Match mapped names against the project's available states. If a mapped state is missing, skip that transition and record a `plane_sync.status_transition_gap`; do not fail the execution result solely because of a missing Plane state.

Plane write policy fields:
- `comment_sync`: create or update a managed progress/final comment.
- `status_sync`: move Plane state using approved mapping.
- `worklog_sync`: create or update one execution worklog.
- `link_sync`: sync evidence, issue candidate, reporter handoff, automation signal, or follow-up refs when tools allow.
- `wiki_sync`: create or update Plane wiki/page. Default is `false`; only enable when the user explicitly requests it or the package says `wiki_sync: true`.
- `create_followup_items`: create Plane follow-up items for failed/blocked issue candidates. Default is `false`.
- `requires_confirmation`: require user confirmation before Plane writes unless a trusted session policy says otherwise.

If write policy is not resolved, produce execution outputs locally and mark Plane sync as `skipped` or `not_started`.

## Notion Preflight

Before any Notion write, resolve write policy and output targets.

Notion write policy fields:
- `source_database_update`: update source test case rows with actual result, status, notes, evidence refs, blockers, retries, and issue candidate refs.
- `execution_page_sync`: create or update one managed execution page. Enable when the user asks for Notion output.
- `execution_database_sync`: create or update execution result database rows.
- `comment_sync`: add or update review/status comments.
- `issue_candidate_sync`: create or update issue candidate pages or rows.
- `requires_confirmation`: require user confirmation before Notion writes unless a trusted session policy says otherwise.

Default Notion policy:
- `execution_page_sync = true` only when the user requests Notion output.
- `source_database_update = false` unless the user asks to update source test cases.
- `execution_database_sync = false` unless the user asks for a result database.
- `comment_sync = false` unless the user asks for comments or review handling.
- `issue_candidate_sync = false` unless the user asks to publish issue candidates.
- `requires_confirmation = true` unless a trusted session policy says otherwise.

If write policy or target page/database/data source is not resolved, produce local execution outputs and mark Notion sync as `skipped` or `not_started`.

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

## Plane Managed Sync

Use one managed Plane comment for progress and final result unless the user requests versioned comments. Include this marker in managed comments:

```html
<!-- qa-executor:execution:<execution_id>:<plane_work_item_id> -->
```

Progress comment content:
- execution id and package id
- run mode
- selected count
- current test case when running
- pass/fail/block/untested/retest counts
- started time
- status mapping summary

Final comment content:
- execution id and package id
- summary counts
- per-test status table
- blockers and retries
- issue candidate refs
- evidence refs
- reporter handoff ref
- automation signal ref
- redaction status
- review status

Status transition rules:
- execution started -> `on_start`
- all selected passed -> `on_all_passed`
- any selected failed -> `on_any_failed`
- any blocked and no failed -> `on_blocked`
- mixed selected/unselected, untested, or incomplete -> `on_partial`
- user cancelled -> `on_cancelled`

Worklog rule: when `worklog_sync = true`, create or update one worklog per `execution_id`. Include package id, selected count, passed/failed/blocked count, run mode, and source Plane readable id in the description.

Link rule: when `link_sync = true`, sync supported refs for evidence, execution result, issue candidates, reporter handoff, automation signal, and created follow-up items. If a link type is unsupported, keep it in the managed comment and record a sync gap.

Wiki rule: create or update a Plane wiki/page only when `wiki_sync = true`. Use `templates/plane-wiki-page.md`. Never update wiki/page by default.

Follow-up rule: failed and blocked cases produce issue candidates by default. Create Plane follow-up work items only when `create_followup_items = true`; link created follow-ups back to the source work item when supported.

Idempotency rule: use `templates/plane-sync-record.json` and `schemas/plane-sync-record.schema.json` to store one record per Plane write action. Use idempotency keys and content hashes to update existing comments, wiki pages, worklogs, links, or follow-ups instead of creating duplicates.

## Notion Managed Sync

Use managed Notion pages, database rows, or comments only when write policy allows them. Include this marker in managed execution pages or comments:

```html
<!-- qa-executor:notion-execution:<execution_id>:<notion_target_id> -->
```

Execution page content:
- execution id and package id
- source Notion page/database refs
- summary counts
- per-test status table
- blockers and retries
- evidence refs
- issue candidate refs
- reporter handoff ref
- automation signal ref
- redaction status
- review status

Source database update rule: when `source_database_update = true`, update existing test case rows only. Do not create duplicate source rows unless the user explicitly asks.

Execution database rule: when `execution_database_sync = true`, create or update result rows keyed by `execution_id + tc_id`. Use `templates/notion-result-row.json` as the row shape.

Comment rule: when `comment_sync = true`, add or update concise comments for execution progress, final result, or OK/NOK review decisions.

Issue candidate rule: when `issue_candidate_sync = true`, create or update issue candidate pages or rows for failed and blocked results. These remain draft candidates, not final submitted defects.

Idempotency rule: use `templates/notion-sync-record.json` and `schemas/notion-sync-record.schema.json` to store one record per Notion write action. Use idempotency keys, managed markers, and content hashes to update existing pages, rows, and comments instead of creating duplicates.

## Retry Policy

Retry only for technical flake: temporary timeout, transient network failure, UI wait or race condition, or temporary service unavailable.

Do not retry business assertion failures. Record every retry in `retries[]` with attempt number, timestamp, reason, outcome, and evidence refs.

## Evidence and Redaction

Capture evidence when available:
- status, actual result, start/finish timestamp, duration, environment, executor identity/tool, step result, assertion result, API summary, UI screenshot/trace/video, DB verification, logs/network refs, retry evidence, blocker detail, and reproduction steps.

Redact sensitive values before output:
- token, password, cookie, credential, secret, authorization value, session id, and PII.

Store large artifacts as refs or paths, not duplicated blobs. Mark missing evidence as a gap instead of inventing it.

Apply the same redaction rules before every Plane write. Do not write raw secrets, raw logs, screenshots with visible PII, or unredacted payloads into Plane comments, worklogs, links, or wiki pages.

Apply the same redaction rules before every Notion write. Do not write raw secrets, raw logs, screenshots with visible PII, or unredacted payloads into Notion pages, database rows, or comments.

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
- keep Plane managed output in sync when Plane write policy allows it.
- keep Notion managed output in sync when Notion write policy allows it.

For `NOK`:
- record feedback in `execution_review_history` using `schemas/execution-review.schema.json` when structured review is needed.
- patch documentation or evidence if it is wrong or incomplete.
- rerun a case only when feedback requires new evidence and environment is safe.
- do not change final status without new evidence or explicit reviewer decision.
- update Plane managed comment/status only when feedback changes execution result or sync output and write policy allows it.
- update Notion managed page/rows/comments only when feedback changes execution result or sync output and write policy allows it.

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
- Plane managed comment using `templates/plane-execution-comment.md` when `comment_sync` is enabled.
- Plane wiki/page using `templates/plane-wiki-page.md` only when `wiki_sync` is enabled.
- Plane sync records using `templates/plane-sync-record.json` when Plane writes are attempted.
- Notion managed execution page using `templates/notion-execution-page.md` when `execution_page_sync` is enabled.
- Notion result rows using `templates/notion-result-row.json` when `execution_database_sync` or `source_database_update` is enabled.
- Notion sync records using `templates/notion-sync-record.json` when Notion writes are attempted.

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

Plane gates, when Plane path is active:
- Plane source was read or `plane_read_gap` was recorded.
- Plane input was executable or `source_not_executable` was recorded.
- status mapping was resolved before status sync.
- write policy was resolved before Plane writes.
- managed comment was synced or sync gap was recorded.
- status transition succeeded or was skipped with reason.
- worklog synced or skipped with reason.
- wiki skipped unless explicitly enabled.
- all Plane writes were redacted.
- duplicate Plane comments, wiki pages, worklogs, links, and follow-up items were avoided.
- Plane managed output matches `execution_result.json`.

Notion gates, when Notion path is active:
- Notion source was read or `notion_read_gap` was recorded.
- Notion input was executable or `source_not_executable` was recorded.
- write policy and output targets were resolved before Notion writes.
- managed execution page, database rows, or comments were synced or sync gaps were recorded.
- source database rows were updated only when `source_database_update = true`.
- issue candidate pages/rows were created only when `issue_candidate_sync = true`.
- all Notion writes were redacted.
- duplicate Notion pages, rows, and comments were avoided.
- Notion managed output matches `execution_result.json`.

## Platform Fallback

Use platform-native API, browser, DB, document, spreadsheet, and JSON tools when available. If tools are unavailable or unsafe, produce manual instructions and structured templates. Core behavior must remain usable in Codex, Claude, and other agent platforms.

## Resources

Read only when needed:
- `schemas/execution-package.schema.json`: normalized input package schema.
- `schemas/execution-result.schema.json`: canonical execution result schema.
- `schemas/execution-review.schema.json`: OK/NOK review feedback schema.
- `schemas/issue-candidate.schema.json`: failed or blocked candidate issue schema.
- `schemas/plane-sync-record.schema.json`: Plane write idempotency and sync result schema.
- `schemas/notion-sync-record.schema.json`: Notion write idempotency and sync result schema.
- `templates/`: reusable result, report, issue, reporter handoff, and automation signal templates.
- `templates/plane-execution-comment.md`: managed Plane execution comment skeleton.
- `templates/plane-wiki-page.md`: opt-in Plane wiki/page skeleton.
- `templates/plane-sync-record.json`: Plane write sync record skeleton.
- `templates/notion-execution-page.md`: managed Notion execution page skeleton.
- `templates/notion-result-row.json`: Notion execution result row skeleton.
- `templates/notion-sync-record.json`: Notion write sync record skeleton.
- `examples/`: manual input normalization and sample execution outputs.
