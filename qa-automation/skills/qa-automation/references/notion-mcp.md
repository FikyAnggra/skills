# Notion MCP Reference

Load this file only when the request includes Notion URLs, page/database/data source ids, Notion MCP payloads, or requests to update automation status in Notion.

## Tool Capabilities

Use Notion MCP tools when available. Tool names vary by client; use the available equivalent and record `notion_mcp_gap` when a required capability is missing.

Read capabilities:
- search workspace content for existing test case sources or automation pages
- fetch pages, databases, data sources, and templates
- query data sources or database views
- read comments/discussions
- resolve users and bot context

Write capabilities:
- create pages
- update pages
- create row pages in databases/data sources
- update row/page properties
- create comments
- create/update managed automation pages when requested
- update schema/data source only when policy allows
- delete/archive only after explicit user approval

## Activation

Notion path is active when any of these exist:
- Notion page URL
- Notion database or data source URL
- page id, database id, or data source id
- MCP payload from Notion
- request to update automation status in Notion
- Plane description/comment/link containing a Notion URL

## Source Status Update

Preserve source locators for every Notion-sourced test case:
- database or data source id
- row/page id
- property map
- source URL
- TC ID

Automation status values:
- `Manual Only`
- `To Be Automate`
- `Automation Implemented`
- `Already Automate`

Update rules:
- Script created or updated -> set `Automation Status` to `Automation Implemented`.
- Script review `OK` plus relevant validation `passed` -> set `Automation Status` to `Already Automate`.
- Validation failed -> keep `Automation Implemented` or previous status, then record validation failure.
- Blocking gap -> keep `To Be Automate` or previous status and record the gap.
- `Manual Only` stays unchanged unless a human explicitly changes it.

Optional fields can be updated only when they exist or schema update policy allows them:
- `Script Path`
- `Validation Status`
- `Review Status`
- `Automation Change Set`
- `Automation Notes`
- `Last Automation Sync`

If a needed field is missing and schema update is unavailable or disabled, record `notion_schema_gap` and use a comment or `Notes` fallback when possible.

## Write Policy

Default policy:
- `source_status_update = true` when a writable test case source is identified
- `comment_sync = true` for concise automation comments when supported/requested
- `automation_page_sync = false` unless requested
- `result_database_sync = false` unless requested
- `schema_update = false` unless requested or explicitly enabled
- `delete_requires_confirmation = true`

Create/update is allowed when request and policy are clear. Delete/archive requires explicit approval after reading the target and summarizing impact.

## Idempotency

Use managed marker:

```text
qa-automation-managed
```

Use idempotency key:

```text
qa-automation:<change_set_id>:notion:<target_type>:<target_id>:<tc_id?>
```

Before creating Notion output, search/fetch for existing managed artifacts by change set id, package id, TC ID, source id, or managed marker. Update managed artifacts instead of creating duplicates.

## Redaction And Fallback

Before writing to Notion, redact secrets, tokens, passwords, cookies, credentials, authorization values, session ids, PII, and sensitive logs.

If Notion tools are missing or fail:
- keep `automation_change_set.json` complete
- generate local Markdown/JSON artifacts
- record `notion_mcp_gap`, `notion_read_gap`, `notion_schema_gap`, or `notion_sync_gap`
- do not claim remote sync succeeded
