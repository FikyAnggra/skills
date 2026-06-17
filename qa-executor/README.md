# QA Executor

`qa-executor` is a portable QA execution skill package. It runs approved test cases when tools and access are safe, produces manual execution instructions when they are not, captures redacted evidence, writes `execution_result.json`, and can sync managed execution output back to Plane.so, Notion, or both through a bridge contract.

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

For Plane Managed Sync, resolve status mapping and write policy before execution. Mapping covers start, all passed, any failed, blocked, partial, and cancelled outcomes. Write policy controls managed comments, status movement, worklogs, links, wiki/page sync, follow-up creation, and confirmation requirements.

Use Notion input when a user provides a Notion page URL, database URL, data source id, page id, or MCP payload. Notion content is executable only when it contains an executor handoff, structured execution package, or minimum test case fields. Raw requirements should be routed to `qa-planner` instead of executed.

For Notion Managed Sync, resolve write policy before writes. Policy controls source database row updates, managed execution page sync, result database rows, comments, issue candidate pages/rows, and confirmation requirements. Notion output is a synced view, not the source of truth.

Use Notion + Plane bridge when Plane tracks the QA work item and Notion stores test cases or execution output. The bridge can discover Notion links from Plane descriptions/comments/links, classify them, and cross-link Plane and Notion rendered outputs while keeping `execution_result.json` canonical. If no Notion URL is found, Notion stays disabled.

Expected outputs are:
- `execution_result.json` as source of truth.
- Markdown execution report.
- spreadsheet-compatible status/result updates.
- issue candidates for failed and blocked cases.
- `qa_reporter_handoff` after human OK review.
- `qa_automation_signal` when automation gaps or update candidates exist.
- managed Plane execution comment when `comment_sync` is enabled.
- Plane worklog/status/link sync when enabled.
- Plane wiki/page only when `wiki_sync` is explicitly true.
- managed Notion execution page when `execution_page_sync` is enabled.
- Notion source row/result row/comment/issue candidate sync when enabled.

## Status Model

Allowed test case statuses are `Untested`, `On Progress`, `Passed`, `Failed`, `Blocked`, and `Retest`. Failed cases need expected vs actual results. Blocked cases need blocker type and reason. Dependency blocks must identify `blocked_by` test case ids.

## Review Loop

Execution output goes through human `OK` or `NOK` review. `OK` accepts the package and enables reporter and automation handoffs. `NOK` records feedback, patches documentation or evidence, and reruns only when new evidence is required and safe.

## Plane Managed Sync

Plane is a synced workflow surface, not the canonical state. `execution_result.json` remains the source of truth.

Managed Plane output uses idempotency keys and a comment marker to avoid duplicate comments, worklogs, wiki pages, links, and follow-up items. Wiki/page sync and follow-up work item creation are off by default. Secrets, tokens, cookies, credentials, authorization values, session ids, and PII must be redacted before any Plane write.

## Notion Managed Sync

Notion is a synced workflow surface, not the canonical state. `execution_result.json` remains the source of truth.

Managed Notion output uses idempotency keys and a managed marker to avoid duplicate pages, rows, and comments. Source database updates, result database sync, comments, and issue candidate publishing are controlled by write policy. Secrets, tokens, cookies, credentials, authorization values, session ids, and PII must be redacted before any Notion write.

## Notion + Plane Bridge

When one execution uses both Plane and Notion, `notion_plane_bridge` records the Plane work item ref, discovered Notion refs, Notion test case source ref, Notion execution page ref, sync direction, cross-links, and bridge idempotency key. Discovered Notion refs are classified as `test_case_source`, `execution_page`, `result_database`, `issue_candidate_database`, or `unknown`. The bridge can add the Notion execution page link to Plane output and the Plane work item link to the Notion execution page. Partial bridge sync does not invalidate the execution result.

Auto Notion reporting can be enabled from discovered links. When exactly one high-confidence Notion `test_case_source` is found in Plane/text, the executor can prepare `source_database_update = true` for databases and `execution_page_sync = true`, while keeping result database, comments, and issue candidate sync off unless requested. Confirmation is still required before writes.

## Downstream Handoff

`qa-reporter` consumes execution summary, evidence refs, issue candidates, coverage/risk refs, and approval metadata. `qa-automation` consumes flaky technical failures, selector/API/data gaps, and automation update candidates. This package does not submit final issues or write final automation scripts.
