---
name: qa-executor
description: Portable QA execution agent for approved qa-planner handoffs, manual test cases, Plane.so work items, Notion pages/databases, spreadsheets, and documents. Executes or guides API/UI/DB/manual QA, enforces guarded Plane workflow states, writes Plane comments only at completion/review by default, captures redacted evidence, writes execution_result JSON, incrementally updates source test case rows/items even when an execution report is requested, embeds or uploads image evidence when supported, syncs managed Plane/Notion output, creates issue candidates, handles OK/NOK review, and produces qa-reporter handoffs and qa-automation signals. Use for QA execution, retest, smoke/targeted runs, evidence capture, source row updates, Plane/Notion execution sync, Plane work item execution, and execution result prep. Do not use for test planning, final defect submission, final automation scripts, or final QA reports.
---

# QA Executor

Use this skill to execute or guide execution of approved QA test cases. Treat `execution_result.json` as the source of truth, then update the original test case source incrementally, render optional reports only when requested, and produce reporter handoffs or automation signals from canonical state.

## Core Rule

Do execution work only. Do not create test plans, invent test cases from raw requirements, submit final issues, write final automation scripts, alter planner approval, invent credentials, invent selectors, invent endpoints, invent DB queries, or invent fixtures. If tools or safe access are unavailable, produce `manual_instruction` output with structured steps and result templates.

Plane, Notion, spreadsheets, documents, and other source artifacts are synced workflow surfaces, not the source of truth. Keep `execution_result.json` canonical and render source row/item updates, Plane/Notion comments, status moves, worklogs, links, wiki pages, and optional report pages from that state.

Default to incremental source updates. Read one test case or a small ordered batch, execute it, update the same source row/item/section when writable, record the update result in `execution_result.json`, then continue. Do not create a new execution report document/page/workbook by default. Create a new report only when the user explicitly asks for report output or the package sets report output policy to enabled. Report creation is additive only; it never replaces source test case status updates.

## Input Paths

Accept four input paths.

Planner-driven input:
- Load `qa_executor` handoff data from `qa-planner`.
- Load approved or explicitly selected test cases, environment constraints, preconditions, test data refs, coverage refs, risk refs, and run policy.
- Prefer planner approval before execution. If the user explicitly requests draft validation, mark source approval as a constraint and avoid implying release-ready execution.

Manual input:
- Accept Excel, Google Sheets, Markdown, Notion, Word/PDF tables, JSON, or plain text test cases.
- Minimum fields are `TC ID`, `Scenario`, `Test Step`, and `Expected Result`.
- Preserve source locators when possible: workbook/sheet row, Google Sheets row, Notion row/page id, document table row, Markdown anchor, JSON pointer, Plane ref, or other stable source pointer.
- Default missing priority to `P2 - Medium`, missing status to `Untested`, run mode to `targeted`, and selected test cases to all provided cases unless the user narrows scope.
- If expected result is missing, do not mark pass/fail. Ask for clarification or produce manual/exploratory instructions with `Blocked` or `Untested` only when justified.

Plane input:
- Accept Plane readable ids such as `ENG-42`, Plane URLs, UUIDs, or MCP payloads.
- Use the Plane guarded execution workflow whenever user input contains Plane context, including `Plane`, `Plane.so`, `work item Plane`, `card Plane`, readable ids such as `DKI-179`, Plane URLs, Plane UUIDs, or requests to read, analyze, execute, review, or update QA results from a Plane work item.
- If input is a readable id such as `DKI-179`, split it into project identifier `DKI` and issue number `179`, then resolve the Plane work item before execution.
- Treat Plane work item/card descriptions, comments, linked refs, wiki/page refs, attachments, cycle/module context, relations, and worklogs as source context when tools allow.
- Scan Plane descriptions, comments, links, and attachment refs for Notion URLs. Store discovered refs in `notion_plane_bridge.discovered_notion_refs` when the bridge is active.
- Read Plane issue detail, project states, comments, description, attachment/link refs, module/cycle, relations, and worklogs when tools allow before deciding executable state.
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
- `plane_state_workflow`: guarded state workflow, current state, expected start state, review state, issue-report state, ready-report state, approval override, and state transition history.

Notion package sections when Notion path is active:
- `notion_context`: source page/database/data source ids, URLs, title, source rows/pages, comments, linked refs, output target refs, and read gaps.
- `notion_write_policy`: source database update, execution page sync, execution database sync, comment sync, issue candidate sync, and confirmation settings.

Cross-source package sections:
- `source_update_policy`: controls incremental updates back to the original test case source.
- `report_output_policy`: controls optional report creation. Default is disabled.
- `evidence_upload_policy`: controls whether screenshots and other visual evidence are embedded/uploaded into writable sources.

Bridge package section when Plane and Notion are both active:
- `notion_plane_bridge`: Plane work item ref, Notion test case source ref, Notion execution page ref, discovered Notion refs, sync direction, source of truth, cross-links, idempotency key, and bridge sync status.

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

## Incremental Source Updates

Default execution loop:
1. Read one test case or a small ordered batch from the source.
2. Execute selected steps.
3. Update `execution_result.json`.
4. Update the same source row/item/section when the source is writable and policy allows it.
5. Continue to the next selected test case.

Do not wait until all tests finish to update source statuses unless the platform only supports batch writes or the user requests a dry run.

Source update fields:
- `Test Case Status`
- `Actual Result`
- `Notes`
- `Evidence Status`
- `Evidence`
- `Executed At`
- `Blocker` or `Issue Candidate Ref` when applicable

Source locator rule: every normalized test case should keep a `source_locator` when discoverable. Use it to update the exact original source without creating duplicates.

Supported locator types:
- Notion database/data source row id or page id.
- Google Sheets spreadsheet id, sheet name, row number, and column map.
- Excel workbook path, sheet name, row number, and column map.
- Google Docs/Word table row or heading anchor.
- Markdown section anchor or table row index.
- JSON pointer.
- Plane work item/comment/wiki ref.

If the source is read-only, ambiguous, or lacks a stable locator, record `source_update_gap` and keep the local `execution_result.json` update only.

## Report Output Policy

Do not create a new execution report page/document/workbook by default.

Create a report only when the user explicitly asks with commands such as `buat report`, `create report`, `publish execution page`, `generate SIT report`, `generate UAT report`, `export Markdown report`, or when `report_output_policy.create_report = true`.

When report output is enabled:
- keep incremental source updates enabled.
- update each selected writable source row/item/section with status and actual result before or alongside report rendering.
- record a `source_update` entry or `source_update_gap` for every selected writable-source test case.
- render the report from `execution_result.json` after source update attempts are recorded.
- never treat a Notion execution page, Google Doc, Word doc, Markdown report, spreadsheet export, Plane wiki, or other report artifact as a substitute for updating the original test case source.

When report output is disabled:
- update source rows/items only.
- keep Plane comments and descriptions summary-only.
- include source links and evidence status refs.
- do not create a Notion execution page solely because a Notion source was discovered.

## Plane Preflight

Before any Plane write, resolve write policy and status mapping.

Plane state workflow:
- `Ready to Test`: work is ready for qa-executor. qa-executor may start directly from this state.
- `In Testing`: qa-executor is actively executing selected test cases.
- `Need Review Test Execute`: qa-executor finished execution and waits for human/user review.
- `Need Issue Report`: review approved and at least one failed, blocked, bug, issue, or problem exists. Send `qa_reporter_handoff` with issue candidates.
- `Ready to Report`: review approved and no failed, blocked, bug, issue, or problem exists. Send `qa_reporter_handoff`.

Default Plane workflow mapping:
- Start execution: `Ready to Test` -> `In Testing`.
- Execution finished: `In Testing` -> `Need Review Test Execute`.
- Review NOK: `Need Review Test Execute` -> `In Testing`.
- Review OK with bug/issue/problem: `Need Review Test Execute` -> `Need Issue Report`.
- Review OK with no bug/issue/problem: `Need Review Test Execute` -> `Ready to Report`.

State validation before execution:
- If the current Plane state is `Ready to Test`, qa-executor may move it to `In Testing` and start execution after normal write-policy approval.
- If the current Plane state is not `Ready to Test`, qa-executor must ask the user for approval before testing or moving the item to `In Testing`.
- Use this prompt shape: `Work item <id> saat ini berada di state <current_state>, bukan Ready to Test. qa-executor hanya boleh langsung bekerja dari Ready to Test. Apakah tetap lanjut testing dan pindahkan ke In Testing?`
- Approval override phrases include `ok`, `approve`, `lanjut`, `ya`, `boleh`, `proceed`, and equivalent confirmations.
- When approval override is given, record the reviewer/user approval text, timestamp, previous state, target state, and reason in `execution_result.json`. Do not create a Plane comment only for approval override; include it later in the completion or review managed comment.
- If approval override is not given, do not execute. Record `plane_state_not_ready` and wait for user action.

Required status mapping keys:
- `on_start`
- `on_all_passed`
- `on_any_failed`
- `on_blocked`
- `on_partial`
- `on_cancelled`

Ask the user for this mapping when it is absent. Match mapped names against the project's available states. If a mapped state is missing, skip that transition and record a `plane_sync.status_transition_gap`; do not fail the execution result solely because of a missing Plane state.

When the project contains the named workflow states above, prefer the Plane state workflow mapping over generic status mapping. If a workflow state is missing from the Plane project, record `plane_sync.status_transition_gap`, ask for mapping only for the missing transition, and continue local execution only when safe.

Plane write policy fields:
- `comment_sync`: create or update a managed Plane comment only when testing is finished or after user review approval. Do not comment on start/progress by default.
- `progress_comment_sync`: create progress comments while execution is running. Default is `false`; enable only when the user explicitly asks for live progress comments.
- `status_sync`: move Plane state using approved mapping.
- `worklog_sync`: create or update one execution worklog.
- `link_sync`: sync evidence, issue candidate, reporter handoff, automation signal, or follow-up refs when tools allow.
- `description_sync`: add or update a managed QA summary block in the Plane work item description. Default is `false`; only enable when the user explicitly requests it or the package says `description_sync: true`.
- `wiki_sync`: create or update Plane wiki/page. Default is `false`; only enable when the user explicitly requests it or the package says `wiki_sync: true`.
- `create_followup_items`: create Plane follow-up items for failed/blocked issue candidates. Default is `false`.
- `requires_confirmation`: require user confirmation before Plane writes unless a trusted session policy says otherwise.

If write policy is not resolved, produce execution outputs locally and mark Plane sync as `skipped` or `not_started`.

## Notion Preflight

Before any Notion write, resolve write policy and output targets.

Notion write policy fields:
- `source_database_update`: update source test case rows with actual result, status, notes, evidence refs, blockers, retries, and issue candidate refs.
- `execution_page_sync`: create or update one managed execution page. Enable only when the user explicitly asks for report output or `report_output_policy.create_report = true`.
- `execution_database_sync`: create or update execution result database rows.
- `comment_sync`: add or update review/status comments.
- `issue_candidate_sync`: create or update issue candidate pages or rows.
- `requires_confirmation`: require user confirmation before Notion writes unless a trusted session policy says otherwise.

Default Notion policy:
- `source_database_update = true` when Notion is the source and writable.
- `execution_page_sync = false` unless the user explicitly requests a new report page.
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

Use one managed Plane comment for completion and review outcomes unless the user requests versioned comments. Do not create or update a Plane comment when moving `Ready to Test` -> `In Testing`, and do not create progress comments while test execution is running unless `progress_comment_sync = true` is explicitly enabled.

Include this marker in managed comments:

```html
<!-- qa-executor:execution:<execution_id>:<plane_work_item_id> -->
```

Completion comment content, written when execution finishes and the state moves `In Testing` -> `Need Review Test Execute`:
- execution id and package id
- summary counts
- failed and blocked TC ids
- blockers and retry summary
- issue candidate refs
- evidence refs
- reporter handoff ref
- automation signal ref
- redaction status
- review status
- review instruction

Review approval comment content, written after user approval when the state moves out of `Need Review Test Execute`:
- reviewer and review decision
- final workflow transition: `Need Review Test Execute` -> `Need Issue Report` or `Ready to Report`
- issue/no-issue decision basis
- qa_reporter_handoff ref
- issue candidate refs when applicable

When a managed Notion execution page exists, keep Plane output summary-only. Do not duplicate the full Notion report in Plane comments, links, worklogs, or description. Plane managed comments may include execution id, package id, run mode, status transition, summary counts, failed and blocked TC ids, blocker count, redaction status, and the Notion execution page link. Omit full per-test tables, detailed steps, detailed evidence, reproduction steps, and full issue candidate details from Plane when Notion is the detailed report surface.

Status transition rules:
- execution started -> `In Testing` when using Plane state workflow, otherwise `on_start`.
- all selected test cases have final status -> `Need Review Test Execute` when using Plane state workflow.
- human review NOK -> `In Testing`.
- human review OK and any failed, blocked, bug, issue, or problem exists -> `Need Issue Report`.
- human review OK and no failed, blocked, bug, issue, or problem exists -> `Ready to Report`.
- user cancelled -> `on_cancelled` or skipped with reason when no cancelled state exists.

During `In Testing`, every selected test case must end as `Passed`, `Failed`, `Blocked`, `Untested`, or `Retest`. Capture actual result, screenshot/trace/video for UI when available, API summary for API, DB verification for DB, logs/network summary when relevant, blocker reason for blocked cases, and redacted evidence refs.

Worklog rule: when `worklog_sync = true`, create or update one worklog per `execution_id`. Include package id, selected count, passed/failed/blocked count, run mode, and source Plane readable id in the description.

Link rule: when `link_sync = true`, sync supported refs for evidence, execution result, issue candidates, reporter handoff, automation signal, and created follow-up items. If a link type is unsupported, keep it in the managed comment and record a sync gap.

Description rule: when `description_sync = true`, update the Plane work item description by inserting or replacing one managed QA summary block. Preserve all user-authored description content outside the managed block. Include this marker in the managed block:

```html
<!-- qa-executor:description-summary:<execution_id>:<plane_work_item_id> -->
```

If a Notion execution page exists, the description block must be summary-only and include the Notion execution page link. Do not place full test tables, detailed evidence, issue candidate details, or reproduction steps in the Plane description.

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

Report coexistence rule: when `execution_page_sync = true` or `report_output_policy.create_report = true`, still update source test case rows/pages when writable and allowed. The execution page is a rendered report, not the test case source update.

Execution database rule: when `execution_database_sync = true`, create or update result rows keyed by `execution_id + tc_id`. Use `templates/notion-result-row.json` as the row shape.

Comment rule: when `comment_sync = true`, add or update concise comments for execution progress, final result, or OK/NOK review decisions.

Issue candidate rule: when `issue_candidate_sync = true`, create or update issue candidate pages or rows for failed and blocked results. These remain draft candidates, not final submitted defects.

Idempotency rule: use `templates/notion-sync-record.json` and `schemas/notion-sync-record.schema.json` to store one record per Notion write action. Use idempotency keys, managed markers, and content hashes to update existing pages, rows, and comments instead of creating duplicates.

## Notion + Plane Bridge

Activate the bridge when the same execution uses both Plane and Notion inputs or outputs.

When Plane is the initial input, scan Plane description, comments, links, and attachment refs for Notion URLs. Classify each discovered Notion ref:
- `test_case_source`: Notion page/database/data source contains executable test case fields.
- `execution_page`: page contains the managed marker `qa-executor:notion-execution` or execution result sections.
- `result_database`: database/data source appears to store execution result rows.
- `issue_candidate_database`: database/data source appears to store issue candidate rows.
- `unknown`: role cannot be determined from title/schema/content.

If exactly one high-confidence `test_case_source` is found, use it as `notion_test_case_source_ref`. If multiple candidate sources or only unknown refs are found, ask the user to choose. If a discovered Notion ref cannot be read, record a `notion_read_gap` and keep the Plane execution valid only when another executable source exists.

Auto Notion source update rule:
- If Plane or user-provided text contains no Notion URL, keep Notion disabled.
- If exactly one high-confidence Notion URL is found and classified as `test_case_source`, enable the bridge and use that URL as the Notion test case source.
- When auto-enabling Notion from a discovered `test_case_source`, set `notion_write_policy.source_database_update = true` when the source is a database/data source. Keep `notion_write_policy.execution_page_sync = false`, `execution_database_sync = false`, `comment_sync = false`, and `issue_candidate_sync = false` unless the user explicitly asks for a report page, result database, comments, or issue candidate publishing.
- Always keep `notion_write_policy.requires_confirmation = true` before writing to Notion.
- If an existing `execution_page` is discovered, update it only when report output is requested. If no execution page is discovered, create a managed Notion execution page only when `execution_page_sync = true`.
- If multiple or ambiguous Notion URLs are found, ask the user to choose the test case source and report target before execution.
- If the discovered Notion URL cannot be read and no other executable source exists, stop and ask for access or clarification.

Bridge source of truth:
- Always use `execution_result.json` as canonical state.
- Treat Notion and Plane as synced views.

Bridge behavior after execution:
- Update Notion test case rows or pages from `execution_result.json` when `notion_write_policy.source_database_update = true`.
- Create or update a Notion execution page only when `notion_write_policy.execution_page_sync = true` or `report_output_policy.create_report = true`.
- If both source updates and a Notion execution page/report are enabled, attempt source row/page updates first or in the same execution pass, then render the execution page from the canonical result.
- Update Plane managed comment, status, worklog, links, and description summary according to `plane_write_policy`. Plane managed comments must follow completion/review-only timing unless `progress_comment_sync = true`.
- Add the Notion execution page link to Plane output when `plane_write_policy.link_sync = true`.
- Add or update the Notion execution page link in the Plane managed comment and managed description summary when those sync policies are enabled.
- Add the Plane work item link to the Notion execution page when `notion_write_policy.execution_page_sync = true`.
- Store discovered Notion links and their classifications in `notion_plane_bridge.discovered_notion_refs`.
- Record whether Notion was auto-enabled from discovered refs in `notion_plane_bridge.auto_notion_report`.

Bridge idempotency key:

```text
qa-executor:<execution_id>:plane:<plane_id>:notion:<notion_id>
```

Bridge sync status:
- `synced`: Plane and Notion bridge writes completed.
- `partial`: one side synced and the other failed or was skipped with a gap.
- `failed`: bridge writes failed on both sides.
- `skipped`: bridge not attempted by policy.

If one side fails, keep the execution valid, set bridge sync status to `partial` or `failed`, and record sync errors. Do not submit final defects, write final reports, or write automation scripts from the bridge.

## Retry Policy

Retry only for technical flake: temporary timeout, transient network failure, UI wait or race condition, or temporary service unavailable.

Do not retry business assertion failures. Record every retry in `retries[]` with attempt number, timestamp, reason, outcome, and evidence refs.

## Evidence and Redaction

Capture evidence when available:
- status, actual result, start/finish timestamp, duration, environment, executor identity/tool, step result, assertion result, API summary, UI screenshot/trace/video, DB verification, logs/network refs, retry evidence, blocker detail, and reproduction steps.

Redact sensitive values before output:
- token, password, cookie, credential, secret, authorization value, session id, and PII.

Store large artifacts as refs or paths, not duplicated blobs. Mark missing evidence as a gap instead of inventing it.

Image evidence policy:
- Prefer embedded/uploaded image evidence in the source system when the target supports file upload, image blocks, image/file properties, or inline media.
- Keep the local artifact path as canonical evidence storage, for example `output/playwright/<work-item>/<tc-id>/screenshot.png`.
- Do not write a local file path as if it were an embedded image in a cloud system. Local paths are fallback refs only.
- For Notion, upload or attach screenshots as image/file evidence when the active Notion MCP/API tools support file upload, image blocks, or files properties. If the tool only accepts external URLs, use an externally accessible URL only when one exists.
- For Google Sheets or Excel, attach or link evidence using supported image/file mechanisms when available; otherwise write an evidence status and local ref.
- For Google Docs, Word, Markdown, and other documents, embed the image only when the document tool supports inserting local images; otherwise write an evidence status and local ref.
- If upload/embed is unavailable, record `evidence_upload_gap` with the target, reason, local path, and intended evidence field.
- Redact screenshots before upload/embed. If redaction cannot be verified, keep the image local and record an evidence gap.

Apply the same redaction rules before every Plane write. Do not write raw secrets, raw logs, screenshots with visible PII, or unredacted payloads into Plane comments, worklogs, links, or wiki pages.

Apply the same redaction rules before every Notion write. Do not write raw secrets, raw logs, screenshots with visible PII, or unredacted payloads into Notion pages, database rows, or comments.

Apply the same redaction rules before writing images or media into spreadsheets, documents, Markdown, JSON-adjacent artifacts, or Plane/Notion surfaces.

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
- when Plane state workflow is active and current state is `Need Review Test Execute`, move Plane to `Need Issue Report` if any failed, blocked, bug, issue, or problem exists; otherwise move Plane to `Ready to Report`.
- after the review OK state transition, create or update the Plane managed comment with the review approval summary when `comment_sync = true`.

For `NOK`:
- record feedback in `execution_review_history` using `schemas/execution-review.schema.json` when structured review is needed.
- patch documentation or evidence if it is wrong or incomplete.
- rerun a case only when feedback requires new evidence and environment is safe.
- do not change final status without new evidence or explicit reviewer decision.
- update Plane status on NOK when write policy allows it. Do not create a Plane comment just for NOK unless the user explicitly asks; record NOK feedback in `execution_result.json`.
- update Notion managed page/rows/comments only when feedback changes execution result or sync output and write policy allows it.
- when Plane state workflow is active and current state is `Need Review Test Execute`, move Plane back to `In Testing`, rerun only feedback-affected test cases when safe, then update `execution_result.json`.
- after rerun completion, create or update the Plane managed comment when the state returns to `Need Review Test Execute`.

## Outputs

Canonical output:
- `execution_result.json` using `schemas/execution-result.schema.json`.

Human outputs:
- Source row/item updates where `Actual Result`, `Test Case Status`, `Notes`, `Evidence`, and `Evidence Status` derive from `execution_result.json`.
- Markdown execution report using `templates/execution-report.md` only when report output is requested.
- Spreadsheet-compatible exports only when requested or required by the source update path.

Downstream outputs:
- `qa_reporter_handoff` using `templates/qa-reporter-handoff.json`.
- `qa_automation_signal` using `templates/qa-automation-signal.json` when relevant.
- Issue candidates using `templates/issue-candidate.json`.
- Plane managed comment using `templates/plane-execution-comment.md` when `comment_sync` is enabled.
- Plane managed description summary using `templates/plane-description-summary.md` when `description_sync` is enabled.
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
- image evidence is uploaded/embedded when supported or `evidence_upload_gap` is recorded.
- every selected writable-source test case has a source update record or `source_update_gap`.
- no new report page/document/workbook is created unless report output is explicitly requested.
- if a report page/document/workbook is created, source update records or gaps still exist for every selected writable-source test case.
- issue candidates exist for `Failed` and `Blocked`.
- source row/item updates and optional rendered reports match `execution_result.json`.
- manual input with missing expected result is not marked pass/fail without clarification.

Plane gates, when Plane path is active:
- Plane source was read or `plane_read_gap` was recorded.
- Plane input was executable or `source_not_executable` was recorded.
- Plane current state was read before execution or `plane_state_read_gap` was recorded.
- Plane project states were read before transition or `plane_state_list_gap` was recorded.
- if current state was not `Ready to Test`, user approval override was captured before execution or execution was skipped with `plane_state_not_ready`.
- status mapping was resolved before status sync.
- write policy was resolved before Plane writes.
- Plane workflow transition used `Ready to Test` -> `In Testing` -> `Need Review Test Execute` -> review outcome state when those states exist.
- no Plane comment was created for `Ready to Test` -> `In Testing` start transition unless `progress_comment_sync = true`.
- no Plane progress comment was created during testing unless `progress_comment_sync = true`.
- managed completion/review comment was synced or sync gap was recorded when testing finished or review approval was processed.
- status transition succeeded or was skipped with reason.
- worklog synced or skipped with reason.
- description summary synced or skipped with reason when `description_sync` is enabled.
- wiki skipped unless explicitly enabled.
- all Plane writes were redacted.
- duplicate Plane comments, wiki pages, worklogs, links, and follow-up items were avoided.
- Plane managed output matches `execution_result.json`.

Notion gates, when Notion path is active:
- Notion source was read or `notion_read_gap` was recorded.
- Notion input was executable or `source_not_executable` was recorded.
- write policy and output targets were resolved before Notion writes.
- source rows/pages were updated incrementally when writable, or sync gaps were recorded.
- managed execution page, database rows, or comments were synced only when requested or sync gaps were recorded.
- source database rows were updated only when `source_database_update = true`.
- execution page was not created unless report output was requested.
- issue candidate pages/rows were created only when `issue_candidate_sync = true`.
- image evidence was uploaded/embedded when Notion tools supported it, otherwise `evidence_upload_gap` was recorded.
- all Notion writes were redacted.
- duplicate Notion pages, rows, and comments were avoided.
- Notion managed output matches `execution_result.json`.

Bridge gates, when both Plane and Notion are active:
- `notion_plane_bridge.source_of_truth` is `execution_result.json`.
- Plane work item ref and Notion source ref are recorded.
- discovered Notion refs from Plane description/comments/links are classified or marked `unknown`.
- Notion execution page link is added to Plane output when Plane link sync is enabled.
- Plane comment and description summary stay summary-only when Notion execution page is the detailed report surface.
- Plane work item link is added to Notion execution page when Notion execution page sync is enabled.
- Bridge idempotency key is recorded.
- Bridge status is `synced`, `partial`, `failed`, or `skipped` with errors when needed.

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
- `templates/plane-description-summary.md`: managed Plane work item description summary block skeleton.
- `templates/plane-wiki-page.md`: opt-in Plane wiki/page skeleton.
- `templates/plane-sync-record.json`: Plane write sync record skeleton.
- `templates/notion-execution-page.md`: managed Notion execution page skeleton.
- `templates/notion-result-row.json`: Notion execution result row skeleton.
- `templates/notion-sync-record.json`: Notion write sync record skeleton.
- `templates/notion-plane-bridge.json`: Notion and Plane cross-sync bridge skeleton.
- `examples/`: manual input normalization and sample execution outputs.
