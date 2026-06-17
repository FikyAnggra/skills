# Notion MCP Reference for QA Reporter

Use this reference when qa-reporter needs to read Notion planner/executor artifacts, publish reporter outputs, process Notion review comments, or sync Notion status from Plane/retest signals.

## Core Rules

- Keep `report_state.json` as the source of truth.
- Treat Notion pages/database rows as source refs or rendered artifacts.
- Fetch database/data source schema before creating or updating database rows.
- Use exact property names from the fetched schema.
- Do not publish secrets, tokens, cookies, credentials, raw PII, or unredacted evidence content.
- If a Notion page or database is inaccessible, record `report_gap` and ask for access or another source.

## Source Reading

Use Notion fetch tools when the user provides a Notion URL/id or when a Plane work item contains Notion links.

Supported source roles:
- `planner_test_cases`
- `planner_report`
- `executor_execution_report`
- `executor_test_case_updates`
- `reporter_publish_target`
- `release_page`
- `unknown`

Read workflow:
1. Fetch the Notion entity by URL or id.
2. If it is a database, identify the data source id from the fetch result.
3. Capture title, url, page id, database id, data source id, schema summary, and role.
4. Extract visible test cases, execution summaries, metrics, issue candidates, evidence refs, or report conclusions only when present.
5. Store source refs in `report_state.source_refs` and details in `report_state.custom_fields.notion.source_pages` or `source_databases`.

## Publishing

Use Notion create/update page tools to publish reporter artifacts when requested.

Publishable artifacts:
- testing report
- issue package
- bug report
- SIT document
- UAT document
- coverage/risk summary
- release sign-off summary

Publishing workflow:
1. Resolve parent page or target data source.
2. Fetch target schema when publishing into a database.
3. Render Markdown from `report_state` and templates.
4. Create Notion page or update existing page.
5. Record page id, url, artifact type, status, timestamp, and source refs in `publication_history`.
6. If publishing fails, keep report artifacts local and record a publication gap.

## Review Comments

Notion comments can drive qa-reporter OK/NOK review.

Review workflow:
1. Fetch page discussions/comments when requested.
2. Normalize reviewer feedback into `reporter-review.schema.json`.
3. Apply feedback to `report_state` first.
4. Re-render impacted artifacts.
5. Reply or comment only when the user asks or when review workflow requires it.

## Status Sync

Before updating Notion status properties, fetch schema and ask for property/status mapping if missing.

Suggested reporter intents for mapping:
- `ready_for_retest`
- `fix_in_progress`
- `retest_passed`
- `retest_failed`
- `closed`
- `needs_review`

Do not assume Notion status option names. Do not treat Plane fix completion as QA passed without retest evidence.

## Plane Link Extraction

When Plane work item descriptions/comments contain Notion URLs:
- extract each URL
- fetch the Notion page/database
- classify the role
- record both Plane and Notion source refs
- use source content only for reporting context and traceability

## Database Property Mapping

Store Notion mapping under `report_state.custom_fields.notion.property_mapping`.

Useful properties when available:
- report id
- project
- feature
- test case id
- test case status
- execution status
- retest status
- QA final status
- sign-off recommendation
- risk level
- Plane work item id
- Plane readable id
- Plane URL
- evidence refs
- last sync timestamp
