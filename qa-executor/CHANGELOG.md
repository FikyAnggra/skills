# Changelog

## 0.5.0 - 2026-06-17

- Added Plane-to-Notion discovery for Notion URLs found in Plane descriptions, comments, links, and attachment refs.
- Added `discovered_notion_refs` to `notion_plane_bridge` schema sections.
- Added Notion ref classifications: `test_case_source`, `execution_page`, `result_database`, `issue_candidate_database`, and `unknown`.
- Updated bridge template and example with discovered Notion refs and read status.
- Migration note: ambiguous or unreadable discovered Notion links should be recorded as gaps and clarified before execution.

## 0.4.0 - 2026-06-17

- Added Notion + Plane bridge contract for executions that use both tools.
- Added `notion_plane_bridge` to execution package and result schemas.
- Added bridge cross-link and idempotency behavior between Plane work items and Notion execution pages.
- Added `notion-plane-bridge.json` template and bridge example.
- Migration note: bridge sync status can be partial while `execution_result.json` remains valid and canonical.

## 0.3.0 - 2026-06-17

- Added Notion page, database, data source, and MCP payload input support for `qa-executor`.
- Added Notion write policy behavior for source database updates, managed execution pages, result database rows, comments, and issue candidate sync.
- Added `notion_context`, `notion_write_policy`, and `notion_sync` schema sections.
- Added `notion-sync-record.schema.json` for idempotent Notion writes.
- Added Notion managed execution page, result row, and sync record templates.
- Added Notion execution package, result, and page examples.
- Migration note: keep Notion output as a synced view of `execution_result.json`; do not treat Notion as canonical execution state.

## 0.2.0 - 2026-06-17

- Added Plane.so work item/card input support for `qa-executor`.
- Added Plane preflight status mapping and write policy behavior.
- Added optional Plane managed comment, status sync, worklog sync, link sync, and opt-in wiki/page sync instructions.
- Added `plane_context`, `plane_write_policy`, `plane_status_mapping`, and `plane_sync` schema sections.
- Added `plane-sync-record.schema.json` for idempotent Plane writes.
- Added Plane managed comment, wiki/page, and sync record templates.
- Added Plane execution package, result, and comment examples.
- Migration note: keep Plane output as a synced view of `execution_result.json`; do not treat Plane as canonical execution state.

## 0.1.0 - 2026-06-11

- Added initial portable `qa-executor` package.
- Added Codex and Claude plugin wrappers.
- Added platform-neutral `qa-executor` skill instructions.
- Added execution package, execution result, execution review, and issue candidate schemas.
- Added result, report, issue candidate, reporter handoff, and automation signal templates.
- Added manual input and execution result examples.
- Compatibility target: Codex skills, Claude-compatible skill wrappers, and platform-neutral agent runtimes.
- Migration note: keep `execution_result.json` as source of truth when future schemas add fields.
