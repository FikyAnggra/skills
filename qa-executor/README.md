# QA Executor

`qa-executor` is a portable QA execution skill package. It runs approved test cases when tools and access are safe, enforces the guarded Plane.so QA state workflow, produces manual execution instructions when needed, captures redacted evidence, writes `execution_result.json`, incrementally updates the original test case source when writable, and can sync managed execution output back to Plane.so, Notion, documents, spreadsheets, or bridge contracts.

## Contents

- `.codex-plugin/plugin.json`: Codex plugin wrapper.
- `.claude-plugin/plugin.json`: Claude-compatible wrapper.
- `skills/qa-executor/SKILL.md`: canonical platform-neutral executor instructions.
- `skills/qa-executor/schemas/`: JSON schemas for execution packages, results, reviews, issue candidates, Plane sync records, and Notion sync records.
- `skills/qa-executor/templates/`: reusable result, report, issue, reporter handoff, automation signal, Plane, Notion, bridge, and sync record templates.
- `skills/qa-executor/examples/`: manual, Plane, and Notion input examples plus sample execution outputs.

## Usage

Use planner-driven input when `qa-planner` has produced a `qa_executor` handoff contract with approved cases, environment constraints, test data refs, and run policy.

Use manual input when a human provides test cases from Excel, Google Sheets, Markdown, Word/PDF tables, Notion, JSON, or plain text. Minimum manual fields are `TC ID`, `Scenario`, `Test Step`, and `Expected Result`.

Use Plane input when a user provides a Plane readable id such as `ENG-42`, a Plane URL, UUID, or MCP payload. Plane content is executable only when it contains an executor handoff, structured execution package, or minimum test case fields. Raw requirements should be routed to `qa-planner` instead of executed.

For Plane Managed Sync, resolve the work item, project states, write policy, and guarded state workflow before execution. qa-executor may start directly only from `Ready to Test`. If the work item is in another state, it must ask for user approval before testing and moving to `In Testing`. Write policy controls completion/review comments, status movement, worklogs, links, wiki/page sync, follow-up creation, and confirmation requirements.

Use Notion input when a user provides a Notion page URL, database URL, data source id, page id, or MCP payload. Notion content is executable only when it contains an executor handoff, structured execution package, or minimum test case fields. Raw requirements should be routed to `qa-planner` instead of executed.

For Notion Managed Sync, resolve write policy before writes. Policy controls source database row updates, managed execution page sync, result database rows, comments, issue candidate pages/rows, and confirmation requirements. Notion output is a synced view, not the source of truth.

Use Notion + Plane bridge when Plane tracks the QA work item and Notion stores test cases or execution output. The bridge can discover Notion links from Plane descriptions/comments/links, classify them, and cross-link Plane and Notion rendered outputs while keeping `execution_result.json` canonical. If no Notion URL is found, Notion stays disabled.

Expected outputs are:
- `execution_result.json` as source of truth.
- incremental source row/item updates for status, actual result, notes, evidence, and evidence status.
- Markdown/Notion/Google Doc/Word/spreadsheet reports only when explicitly requested.
- issue candidates for failed and blocked cases.
- `qa_reporter_handoff` after human OK review.
- `qa_automation_signal` when automation gaps or update candidates exist.
- managed Plane execution comment when testing is finished or review is approved and `comment_sync` is enabled.
- Plane worklog/status/link sync when enabled.
- Plane wiki/page only when `wiki_sync` is explicitly true.
- managed Notion execution page only when `execution_page_sync` is enabled by an explicit report request.
- Notion source row/result row/comment/issue candidate sync when enabled.

## Incremental Source Updates

By default, `qa-executor` reads one test case row/item or a small ordered batch, executes it, writes the result to `execution_result.json`, then updates the same source row/item/section before moving to the next case. Supported writable sources include Notion rows/pages, Plane items, Google Sheets, Excel, Google Docs, Word/PDF-derived tables where editable output is available, Markdown, and JSON.

New report documents are opt-in. The executor creates a report page/document/workbook only when the user explicitly asks for it or `report_output_policy.create_report = true`.

Evidence images are embedded or uploaded into the source when the active tool supports image/file upload or inline image insertion. Local screenshot paths such as `output/playwright/DKI-140/...` remain canonical local evidence refs. If a cloud tool cannot upload or embed local images, qa-executor records `evidence_upload_gap` instead of pretending the local path is an embedded image.

## Status Model

Allowed test case statuses are `Untested`, `On Progress`, `Passed`, `Failed`, `Blocked`, and `Retest`. Failed cases need expected vs actual results. Blocked cases need blocker type and reason. Dependency blocks must identify `blocked_by` test case ids.

## Review Loop

Execution output goes through human `OK` or `NOK` review. `OK` accepts the package and enables reporter and automation handoffs. `NOK` records feedback, patches documentation or evidence, and reruns only when new evidence is required and safe.

## Plane Managed Sync

Plane is a synced workflow surface, not the canonical state. `execution_result.json` remains the source of truth.

Managed Plane output uses idempotency keys and markers to avoid duplicate comments, description summaries, worklogs, wiki pages, links, and follow-up items. By default, qa-executor does not create comments when moving `Ready to Test` -> `In Testing` and does not create progress comments while testing. Plane comments are written only when testing finishes or after user review approval, unless live progress comments are explicitly enabled. Wiki/page sync, description sync, and follow-up work item creation are off by default unless the write policy enables them. Secrets, tokens, cookies, credentials, authorization values, session ids, and PII must be redacted before any Plane write.

Guarded Plane workflow:
- `Ready to Test` -> `In Testing` when execution starts.
- `In Testing` -> `Need Review Test Execute` when all selected test cases have final status.
- `Need Review Test Execute` -> `In Testing` on NOK review.
- `Need Review Test Execute` -> `Need Issue Report` on OK review with any failed, blocked, bug, issue, or problem.
- `Need Review Test Execute` -> `Ready to Report` on OK review with no issue.

If the initial state is not `Ready to Test`, qa-executor must ask for approval before execution and record the approval override in `execution_result.json` and the managed Plane comment.

## Notion Managed Sync

Notion is a synced workflow surface, not the canonical state. `execution_result.json` remains the source of truth.

Managed Notion output uses idempotency keys and a managed marker to avoid duplicate pages, rows, and comments. Source database updates, result database sync, comments, and issue candidate publishing are controlled by write policy. Secrets, tokens, cookies, credentials, authorization values, session ids, and PII must be redacted before any Notion write.

## Notion + Plane Bridge

When one execution uses both Plane and Notion, `notion_plane_bridge` records the Plane work item ref, discovered Notion refs, Notion test case source ref, Notion execution page ref, sync direction, cross-links, and bridge idempotency key. Discovered Notion refs are classified as `test_case_source`, `execution_page`, `result_database`, `issue_candidate_database`, or `unknown`. The bridge can add the Notion execution page link to Plane output and the Plane work item link to the Notion execution page. Partial bridge sync does not invalidate the execution result.

Auto Notion source updates can be enabled from discovered links. When exactly one high-confidence Notion `test_case_source` is found in Plane/text, the executor can prepare `source_database_update = true` for databases while keeping execution page, result database, comments, and issue candidate sync off unless requested. Confirmation is still required before writes.

Default Notion behavior is source row update, not report creation. When a Notion test case database/page is the source, qa-executor updates status, actual result, notes, evidence status, and evidence on the original row/page. A separate Notion execution page is created only when requested.

When a Notion execution page exists, Plane should remain a workflow surface. Plane managed comments and description summary blocks should include only summary counts, failed or blocked TC ids, status, and the Notion execution page link. Full test tables, detailed evidence, reproduction steps, and issue candidate details stay in Notion.

## Downstream Handoff

`qa-reporter` consumes execution summary, evidence refs, issue candidates, coverage/risk refs, and approval metadata. `qa-automation` consumes flaky technical failures, selector/API/data gaps, and automation update candidates. This package does not submit final issues or write final automation scripts.
