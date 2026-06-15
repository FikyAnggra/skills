# Notion MCP Reference

Load this file only when input or requested output contains Notion signals such as a Notion page/database URL, request to create a Notion test plan, request to create test cases in Notion, or request to copy/share a Notion link.

## Notion MCP Tool Map

Use Notion MCP tools when available. Tool names are based on Notion MCP docs; in OpenAI MCP clients, `notion-search` and `notion-fetch` may appear as `search` and `fetch`.

Intake and review tools:
- `notion-search`: find existing requirement pages, QA plans, test case databases, or parent destination pages.
- `notion-fetch`: read a Notion page, database, data source schema, template, or existing QA artifact by URL or ID.
- `notion-get-comments`: read review comments and discussions on a Notion page before revising planning output.
- `notion-query-data-sources`: query existing Notion data sources when the workspace supports it, for example to inspect existing QA cases or requirements grouped by status/owner.
- `notion-query-database-view`: query a predefined database view when data source querying is unavailable but a QA database view exists.
- `notion-get-users`, `notion-get-user`, `notion-get-self`: resolve owners, reviewers, and connected workspace/bot context when available.

Output tools:
- `notion-create-pages`: create the QA test plan page and create test case row pages inside the test case database.
- `notion-update-page`: update managed sections of an existing test plan page or test case row page.
- `notion-create-database`: create the Notion test case database. This is mandatory for `test_cases` when Notion output is requested and the tool is available.
- `notion-update-data-source`: update the test case database/data source schema when required properties are missing.
- `notion-create-view`: create useful database views such as All Cases, By Priority, By Status, Automation Candidates, and Manual Only.
- `notion-update-view`: update existing views when rerunning a managed QA package.
- `notion-create-comment`: add review/status comments to the test plan page when requested or useful for review handoff.

Rate behavior:
- Avoid high-parallel Notion calls. Keep search calls low; Notion MCP docs list search as stricter than general request limits.
- For large test suites, create database schema first, then create row pages in bounded batches.
- If rate-limited, record `notion_rate_limit_gap`, stop before partial duplication, and ask the user whether to resume later.

## Output Model

When Notion output is requested:
- Create one Notion page for the test plan.
- Create one Notion database for test cases. Do not use a plain table page unless `notion-create-database` is unavailable or the user explicitly requests page-only fallback.
- Create each test case as a database row/page using `notion-create-pages` with database properties and detailed body content.
- Store created or reused Notion URLs and IDs in `planning_state.artifact_outputs`, `planning_state.notion_context`, and relevant handoff contracts.

Required test case database properties:
- `TC ID`: title or rich text, stable test case id such as `TC0001`
- `Scenario`: rich text
- `Summary`: rich text
- `Test Type`: select with qa-planner allowed `test_type` values
- `Priority`: select with qa-planner allowed priority values
- `Status`: select with qa-planner allowed test case status values
- `Automation Status`: select with qa-planner allowed automation status values
- `Requirement Refs`: multi-select or rich text
- `Owner`: people or rich text
- `Last Updated`: date

Recommended database properties:
- `Package ID`: rich text
- `Planning Status`: select
- `Source`: URL or rich text
- `Tags`: multi-select
- `Execution Link`: URL

Test case row/page body must include long fields that do not fit cleanly as properties:
- preconditions
- ordered test steps
- test data
- expected result
- notes/gaps
- source refs and requirement refs

Required test plan page sections:
- package summary and version
- source refs
- scope and exclusions
- risks
- test strategy
- coverage summary
- Notion test case database link
- open questions and gaps
- handoff links
- changelog

## Link Handling

Always capture shareable Notion links returned by tools or available in fetched payloads.

Store links in:
- `planning_state.artifact_outputs[]` with `format = notion`
- `planning_state.notion_context.test_plan_page_url`
- `planning_state.notion_context.test_case_database_url`
- `handoff_contracts.*.source_refs.notion_artifacts[]`

When another platform is active, such as Plane, include Notion links in that platform's sync output instead of duplicating full Notion content.

If Notion URL is unavailable but page/database ID exists, store the ID and record `notion_link_gap`.

## Idempotency

Before creating new Notion output:
- Search/fetch for existing managed artifacts by `package_id`, source requirement id, source Plane id, or page title.
- Reuse and update existing managed page/database when found.
- Create a new page/database only when no matching managed artifact exists or the user requests a new version.

Managed marker: `qa-planner-managed`.

Never create duplicate databases for the same package id unless versioning is explicitly requested.

## Database Creation Flow

1. Resolve destination parent page/database using user input or `notion-search`.
2. If destination is missing, ask the user for destination unless private-page creation is explicitly acceptable.
3. Create or reuse the test plan page with `notion-create-pages` / `notion-update-page`.
4. Create or reuse the test case database with `notion-create-database`.
5. Ensure required database properties exist with `notion-fetch` and `notion-update-data-source` when needed.
6. Create/update views with `notion-create-view` or `notion-update-view`.
7. Create test case row pages with `notion-create-pages` in batches.
8. Update the test plan page with the database link and summary counts.
9. Record all URLs/IDs in `planning_state` and handoff contracts.

## Fallback

If `notion-create-database` is unavailable:
- Ask the user whether to continue with Markdown/JSON output or page-only fallback.
- Page-only fallback may render a Markdown table inside the test plan page, but must mark `test_cases_database_status = unavailable` and record `notion_database_gap`.

If Notion write tools are unavailable:
- Generate local JSON/Markdown artifacts.
- Record `notion_sync_gap`.
- Do not claim that Notion output was created.

## Safety

Before writing to Notion, redact or ask before publishing secrets, credentials, tokens, cookies, private keys, PII, sensitive test accounts, internal incident notes, or unpublished customer data.

Do not make public sharing claims unless the tool response confirms the sharing scope or the user explicitly configures it outside this skill.
