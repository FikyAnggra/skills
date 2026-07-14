---
name: qa-planner
description: Portable QA planning agent for converting requirements, PRDs, user stories, API specs, UI flows, screenshots, database notes, technical constraints, optional Plane.so work items/cards/sub-work-items, and optional Notion pages/databases into governed planning_state JSON, test plans, separated UI/API test cases, coverage maps, OK/NOK or partial review updates, default document-style test plans with Test Case and Handoff attachments, Notion test plan pages, Notion UI/API test case databases, copied Notion links, Plane parent/sub-work-item comments, and handoff contracts for qa-executor, qa-automation, and qa-reporter. Use when asked to create or revise QA plans, QA scenarios, UI test cases, API test cases, requirement coverage, downstream QA handoffs, Plane-attached planning artifacts, Notion-published planning artifacts, or planning review packages. Do not use for running tests, creating defect tickets, writing final automation scripts, or producing final execution reports.
---

# QA Planner

Use this skill to create and revise QA planning artifacts. Treat `planning_state.json` as the source of truth, then render human-readable plans, test cases, coverage maps, and handoff contracts from that state.

## Core Rule

Do planning work only. Do not execute tests, create defect tickets, produce final execution reports, write final automation scripts, invent selectors, invent credentials, invent endpoints, invent fixture data, or change execution statuses without verified executor results or explicit user instruction.

Do not use assumptions for missing, ambiguous, or conflicting source information that affects scope, expected results, data, priority, automation feasibility, Plane write targets, or downstream handoff behavior. Ask the user or record a blocking `open_question` before planning those parts.

## Input Intake

Accept raw or structured inputs:
- requirements, PRDs, user stories, acceptance criteria, business rules, UI flows, screenshots, API specs, database notes, architecture notes, constraints, risks, and release scope
- existing test cases or QA plans that need normalization or revision
- human review feedback with `OK`, `NOK`, or partial target feedback
- Plane.so work items/cards, sub-work-items, comments, readable attachments, pages/wiki, modules, cycles, parent/child items, or linked items
- Notion pages/databases, Notion output destinations, or requests to publish/copy/share Notion links for test plans and test cases

If the input contains Plane.so signals such as `Plane`, a Plane work item/card/sub-work-item, a readable id like `ENG-42`, a Plane URL/UUID/payload, or an @mention-style Plane Agent request, read `references/plane-hybrid.md` before continuing.

If the input or requested output contains Notion signals such as a Notion URL, page/database id, request to create a Notion test plan, request to create test cases in Notion, or request to copy a Notion link, read `references/notion-mcp.md` before continuing.

If sources conflict, do not choose silently. Add an `open_question` that names both sources and their conflicting values, then ask the user which source wins before planning impacted cases.

Blocking ambiguity, ask before planning impacted scope:
- scope boundary changes what is tested
- expected result cannot be derived from source material
- target environment, platform, API version, or user role changes behavior
- priority, required validation rule, or business rule is invalid or contradictory
- fixture/test data is required to design meaningful steps

Non-blocking ambiguity, record in `open_questions` and continue:
- wording polish that does not change behavior
- minor field label differences where expected result stays identical
- step order detail that does not change user-visible outcome
- missing owner/date metadata

## Planning State

Create or update `planning_state.json` before rendering outputs. Use `schemas/planning-state.schema.json` when validation tooling exists.

Required sections:
- `metadata`: project, feature, owner, version, planning status, created date, updated date
- `source_inputs`: normalized source references and source summaries
- `assumptions`: explicit user-approved assumptions only
- `open_questions`: missing, ambiguous, or conflicting information
- `scope`: in-scope and out-of-scope items
- `risk_analysis`: product, technical, data, integration, operational, and business risks
- `test_strategy`: test levels, test types, priority model, coverage target, execution approach
- `test_plan`: objective, environment, entry criteria, exit criteria, schedule notes, dependencies
- `ui_test_cases`: UI/manual-flow test cases using the UI field model
- `api_test_cases`: API/contract test cases using the API field model
- `test_cases`: legacy aggregate for compatibility; keep in sync with `ui_test_cases` plus API summaries when downstream tools only support one list
- `coverage_map`: requirement-to-test-case mapping or exclusion reason
- `handoff_contracts`: contracts for `qa-executor`, `qa-automation`, and `qa-reporter`
- `review_history`: OK/NOK/partial feedback and resolution history
- `changelog`: versioned change summary

Allowed `planning_status` values:
- `draft`
- `revision_required`
- `approved`

Inline fallback skeleton when schema files are unavailable:

```json
{
  "metadata": {
    "package_id": "QA-PLAN-<PROJECT>-<FEATURE>-v0.1",
    "project": "",
    "feature": "",
    "owner": "",
    "version": "0.1",
    "planning_status": "draft",
    "created_at": "",
    "updated_at": ""
  },
  "source_inputs": [],
  "assumptions": [],
  "open_questions": [],
  "scope": { "in_scope": [], "out_of_scope": [] },
  "risk_analysis": [],
  "test_strategy": {},
  "test_plan": {},
  "ui_test_cases": [],
  "api_test_cases": [],
  "test_cases": [],
  "coverage_map": [],
  "handoff_contracts": { "qa_executor": {}, "qa_automation": {}, "qa_reporter": {} },
  "plane_context": {
    "parent_work_item": {},
    "sub_work_items": [],
    "processed_work_items": []
  },
  "review_history": [],
  "changelog": []
}
```

## Workflow

Choose one path first.

Before doing the requested planning task, run a requirement gap check:
1. Read the user's request and available source inputs.
2. List required-but-missing information needed to complete the requested task safely, such as scope, acceptance criteria, expected results, target environment, user roles, platform/API version, fixture data, credentials, endpoints, priority rules, output destination, Plane/Notion write target, or downstream handoff target.
3. Classify each missing item as `blocking` or `non_blocking` using the ambiguity rules above.
4. Ask the user for blocking missing information before planning impacted scope.
5. Record non-blocking missing information in `open_questions` and continue only for scope that can be planned safely.

Standard path:
1. Intake source inputs and identify source refs.
2. Perform the requirement gap check and show the user the missing/needed information list before continuing.
3. Normalize inputs into requirements, feature map, assumptions, and open questions.
4. Classify ambiguity as blocking or non-blocking using the rules above; ask only for blocking ambiguity and record non-blocking ambiguity in `open_questions`.
5. Analyze ambiguity, gaps, conflicts, edge cases, state transitions, validation gaps, risks, and dependencies.
6. Build test strategy and test plan content.
7. Classify and design test cases into `ui_test_cases` and `api_test_cases` using the approved split field models.
8. Build coverage map.
9. Render artifacts using the output decision rules.
10. Apply review feedback until the package is approved or remaining gaps are explicit.

Plane path:
1. Read `references/plane-hybrid.md`.
2. Resolve Plane source refs and read the work item/card plus readable attachments when tools allow.
3. If the request includes parent plus sub-work-items, resolve and read the parent first, then each unique requested sub-work-item; preserve parent-child context in `planning_state.plane_context`.
4. Apply the Plane QA state workflow before analysis or generation for every requested work item and sub-work-item. Default eligible source state is `Todo Test`; any other source state requires a separate user confirmation after the current state is known. A direct item id or direct planning request is not confirmation.
5. Move `Todo Test` to `Analyze Test` when starting analysis, route `Analyze Test` to `Ready to Test` or `Update Test`, route `Update Test` to `Need Review Test`, and route review feedback according to the Plane state machine.
6. Continue with the standard path only after the state gate allows work for the impacted item(s).
7. Sync outputs back to the source Plane work item/card and every processed sub-work-item according to the Plane output policy.

Notion path:
1. Read `references/notion-mcp.md`.
2. Resolve destination parent page/database or ask the user when no destination is clear.
3. Continue with the standard path.
4. Create or update a Notion test plan page.
5. Create or update separate Notion UI/API test case databases as needed. UI and API cases must use their exact database columns when `notion-create-database` and required `notion-update-data-source` support are available.
6. Store Notion page/database URLs in `planning_state.artifact_outputs`, `planning_state.notion_context`, and handoff contracts.

Large-input handling:
- If estimated test cases exceed 50, propose batching by feature, module, epic, API group, or risk priority before generating all cases.
- If estimated test cases exceed 100 or source input is too large to hold safely, ask the user to choose first batch scope.
- Preserve one `planning_state` and append batches into `ui_test_cases`, `api_test_cases`, legacy `test_cases`, `coverage_map`, `review_history`, and `changelog`.
- Never truncate silently. Record omitted scope explicitly.

## Test Case Field Models

Split generated cases by execution surface. Do not mix API request fields into UI cases or UI step fields into API cases.

Classification:
- Use `api_test_cases` when source evidence mentions endpoint, HTTP method, URL/path, headers, params, request body, response body, status code, API contract, webhook, integration API, auth token, or service-to-service behavior.
- Use `ui_test_cases` when source evidence mentions screen, button, form, navigation, copy, layout, visual behavior, responsive behavior, browser, or user-facing workflow.
- If one requirement needs both API and UI coverage, create separate UI and API cases and map both IDs to the same requirement in `coverage_map`.
- If the surface is unclear and affects test design, ask the user before generating impacted cases.

### UI Test Case Model

Use `templates/Template Test Case.xlsx` as the UI spreadsheet baseline. Markdown, Word, PDF, Notion, and JSON outputs must preserve these logical fields for UI cases:

| Field | Rule |
|---|---|
| `tc_id` | Stable identifier such as `TC0001` |
| `scenario` | Short scenario label, for example `Login` or `Apply promo code` |
| `summary` | Test purpose, for example `Verify valid user signs in and lands on dashboard` |
| `test_type` | Use only allowed test type values |
| `priority` | Use only allowed priority values |
| `pre_conditions` | Preconditions before execution |
| `test_steps` | Ordered steps |
| `test_data` | Input, fixture, account, payload, or dataset refs |
| `expected_result` | Clear observable outcome |
| `actual_result` | Blank before execution |
| `test_case_status` | Latest execution status |
| `automation_status` | Automation treatment |
| `notes` | Planner notes, constraints, or gaps |

### API Test Case Model

Use `templates/test-case-api.md` as the API baseline. Markdown, Word, PDF, Notion, and JSON outputs must preserve these logical fields for API cases:

| Field | Rule |
|---|---|
| `tc_id` | Stable identifier such as `TC0001`; do not reuse an ID from UI cases |
| `scenario` | Short API scenario label, for example `Create user` or `Reject invalid token` |
| `summary` | Test purpose, for example `Verify POST /users returns 201 for a valid payload` |
| `method` | HTTP method: `GET`, `POST`, `PUT`, `PATCH`, `DELETE`, `HEAD`, or `OPTIONS` |
| `url` | Endpoint path or URL; do not invent base URL when source only provides path |
| `header` | Required request headers, auth scheme, content type, or header refs |
| `params` | Query/path params and values or fixture refs |
| `body` | Request body, payload shape, or fixture ref |
| `expected_result` | Expected status code, response body, schema, side effect, or error contract |
| `actual_result` | Blank before execution |
| `test_case_status` | Latest execution status |
| `automation_status` | Automation treatment |
| `notes` | Planner notes, constraints, contract refs, or gaps |

Allowed priority values:
- `P0 - Critical`
- `P1 - High`
- `P2 - Medium`
- `P3 - Low`

Allowed test case status values:
- `Untested`
- `On Progress`
- `Passed`
- `Failed`
- `Blocked`
- `Retest`

Allowed automation status values:
- `Manual Only`
- `To Be Automate`
- `Automation Implemented`
- `Already Automate`

Allowed test type values:
- `Functional`
- `API`
- `UI`
- `E2E`
- `Integration`
- `Regression`
- `Smoke`
- `Negative`
- `Boundary`
- `Accessibility`
- `Performance`
- `Security`
- `Usability`

If a source uses a non-listed type such as `Happy Path` or `Sanity`, map it to the closest allowed type only when the mapping is obvious and note the mapping. If not obvious, ask the user.

Automation status decision rules:
- `Already Automate`: choose only when source references automation that has been reviewed, approved, and maintained as existing coverage, such as an approved suite, test id, CI job, or maintained automation coverage.
- `Automation Implemented`: choose when an automation script has already been created or implemented, but is not yet reviewed, not yet approved, or not yet proven as stable maintained coverage.
- `To Be Automate`: choose for repetitive regression candidates with stable UI/API behavior, deterministic data, observable expected result, and known or discoverable automation surface.
- `Manual Only`: choose for exploratory testing, subjective visual judgment, usability review, one-off investigation, unsupported external dependency, unstable selectors, missing fixtures, CAPTCHA/OTP/manual approval, or high automation cost compared with regression value.
- Borderline cases default to `Manual Only` plus a note explaining what information would make automation feasible.

Rules:
- New test cases default to `test_case_status = Untested`.
- `actual_result` stays blank until `qa-executor` returns a result or user explicitly provides one.
- If input priority is not one of the allowed values, add an `open_question` and ask the user to clarify. Do not downgrade or remap invalid priority silently.
- Apply the automation status decision rules above; when evidence is insufficient, use `Manual Only` and record the gap.
- Mark missing selectors, endpoints, methods, headers, params, request body, fixture data, or credentials as gaps, not invented data.
- When Plane parent/sub-work-item context is active, prefix `scenario` with the readable work item id that sourced the case, for example `DKI-179 - Login validation`. Apply the same rule to parent-derived cases and sub-work-item-derived cases.

## Coverage Map

Always map each requirement to test cases or an exclusion reason.

Minimum Markdown format:

| Requirement ID | Requirement Summary | Test Case IDs | Coverage Status | Notes |
|---|---|---|---|---|
| REQ-01 | Login valid user | TC0001, TC0002 | Covered | |
| REQ-02 | Password reset | - | Excluded | Out of sprint scope |

Allowed coverage status values: `Covered`, `Partial`, `Gap`, `Excluded`.

## Review Loop

Use one review model for all sources, including Plane comments. Use `schemas/review-feedback.schema.json` when structured review feedback is needed.

| Review input | Action | Planning status |
|---|---|---|
| `OK` | Record approval, lock current package version, make handoff contracts ready | `approved` |
| `NOK: <feedback>` | Record issue, revise impacted sections, re-render impacted artifacts | `revision_required` |
| `scope only` | Treat scope as approved, keep other sections open for review | keep current or `revision_required` |
| `test cases need revision` | Revise only impacted test cases and coverage map | `revision_required` |
| `regenerate cases` | Regenerate test cases from approved scope and source refs | `revision_required` |
| `update wiki` or Plane-specific command | Follow Plane reference, then update managed Plane output | keep current unless feedback changes planning |

For partial feedback, preserve approved sections and revise only targeted sections unless the feedback exposes a broader contradiction.

## Handoff Contracts

Use `schemas/handoff-contract.schema.json` for each downstream contract when validation tooling exists. Generate contracts from `planning_state` only.

Generate handoff contracts when at least one is true:
- `planning_status = approved`
- user explicitly requests downstream handoff
- output is a review-ready draft and the contract is clearly marked `draft`

Do not mark contracts ready for execution, automation, or reporting before approval unless the user explicitly requests draft validation.

Inline fallback contract shape when schema files are unavailable:

```json
{
  "contract_version": "1.0",
  "source_agent": "qa-planner",
  "target_agent": "qa-executor",
  "package_id": "QA-PLAN-...",
  "approval_status": "draft",
  "source_refs": {
    "planning_state": "planning_state.json",
    "test_cases": "planning_state.test_cases",
    "ui_test_cases": "planning_state.ui_test_cases",
    "api_test_cases": "planning_state.api_test_cases",
    "coverage_map": "planning_state.coverage_map"
  },
  "scope": {
    "candidate_test_cases": [],
    "selected_test_cases": []
  },
  "constraints": [],
  "gaps": []
}
```

For `qa-automation`, set `target_agent = qa-automation` and use scope fields `create_scripts`, `update_scripts`, and `manual_only`. For `qa-reporter`, set `target_agent = qa-reporter` and use scope fields `coverage_refs`, `risk_refs`, and `candidate_test_cases`.

## Output Decision Rules

Always produce or update `planning_state.json` as the canonical state.

Default local/document output when no Plane, Notion, spreadsheet, document, or other platform target is provided:
1. Render a test plan document using this section order: title `Test Plan Name`, objective, source inputs, scope with `In Scope` and `Out of Scope`, assumptions and open questions, test strategy, entry criteria, exit criteria, and `Attachment`.
2. Under `Attachment`, include `Test Case` and `Handoff`.
3. Prefer creating `Test Case` as a separate document artifact attached or linked from the test plan. If separate document generation is unavailable, render `Attachment -> Test Case` as tables inside the test plan. If tables are unavailable, render it as structured text.
4. Prefer creating `Handoff` as a separate document artifact attached or linked from the test plan. If separate document generation is unavailable, render `Attachment -> Handoff` as tables inside the test plan. If tables are unavailable, render it as structured text.

Produce additional artifacts only under these rules:
- Test plan: produce for new planning packages or when plan sections changed.
- Test cases: produce separated UI and API artifacts when requirements, scope, risks, or cases changed.
- Coverage map: produce when requirements or test cases changed.
- Handoff contracts: produce only when approved, explicitly requested, or clearly marked draft.
- Spreadsheet: produce when user asks for Excel/Sheets, when Plane/full-sync policy requests attachments, or when test cases are large enough that a table is hard to review in Markdown.
- Plane comment/page/sync outputs: produce only when Plane path is active and Plane write policy allows it.
- Notion page/database outputs: produce only when Notion path is active and Notion write policy allows it. Test plan output is a page rendered from `templates/test-plan.md`; UI test cases use a UI test case database with exact `templates/test-case.md` columns, and API test cases use an API test case database with exact `templates/test-case-api.md` columns. If database/schema tooling is unavailable, create a Notion page containing separate UI and API tables as needed.

If schemas or templates are unavailable, generate inline JSON/Markdown structures from the field model and rules in this skill. Do not fail solely because `/schemas/` or `/templates/` files are unavailable.

## Quality Gates

Content gates, always apply:
- No unresolved critical missing or conflicting source information without an `open_question`.
- Priority values match the allowed enum exactly.
- New test cases have `test_case_status = Untested`.
- Automation status values match the allowed enum exactly.
- API and UI cases are separated into `api_test_cases` and `ui_test_cases`; mixed-surface requirements map to both case types instead of one overloaded case.
- API cases include method, URL/path, headers, params, body, and expected result from source evidence or record a blocking gap/open question.
- Each test case has a clear expected result.
- Each requirement maps to at least one test case or has an exclusion reason.
- Duplicated or overlapping cases are removed, merged, or noted.
- Rendered human artifacts match `planning_state`.
- Default non-platform output includes a test plan document with `Attachment -> Test Case` and `Attachment -> Handoff`, using separate document artifacts first, embedded tables second, and structured text only as final fallback.

Plane sync gates, apply only on Plane path:
- Plane QA state workflow was applied before analysis, generation, or sync.
- Plane source state was `Todo Test`, or separate user confirmation for the non-`Todo Test` state was recorded after state lookup. For parent/sub-work-item requests, this applies to each processed item.
- `Analyze Test` was used for analysis/routing only, not large artifact creation.
- `Update Test` was used for creating/updating requested artifacts.
- Plane MCP context was resolved when tools were available: work item payload, comments, activities, relations, states, members, properties, types, and requested cycle/module scope.
- Parent/sub-work-item requests resolved the parent first, deduplicated requested sub-work-item ids, preserved parent-child refs, and created test case scenarios with readable id prefixes.
- Existing Plane pages/wiki relevant to the request were read before use or `plane_page_read_gap` was recorded.
- Existing managed Plane pages were updated only inside managed sections when update tooling existed; otherwise a versioned page was created and `plane_page_update_gap` was recorded.
- Readable attachment content was read or `attachment_read_gap` was recorded.
- Plane output was added or updated on the source work item/card when write tools were available.
- For each processed sub-work-item, a managed result comment was added or updated on that sub-work-item when write tools were available; otherwise `plane_sub_work_item_comment_gap` was recorded.
- Managed Plane output avoids duplicate comments/pages/attachments.
- Plane handoff tracker work items are created only when requested or required by an approved handoff contract.
- Sensitive data is redacted before Plane write.

Notion sync gates, apply only on Notion path:
- Destination parent page/database was resolved or the user approved fallback.
- Test plan was created or updated as a Notion page when write tools were available.
- Test plan page follows the structure of `templates/test-plan.md` and avoids raw JSON dumps.
- UI and API test cases were created or updated as separate Notion databases when both case types exist and `notion-create-database` plus required `notion-update-data-source` support were available.
- UI test case database has exact Notion display columns in this order: `TC ID`, `Scenario`, `Summary`, `Test Type`, `Priority`, `PreConditions`, `Test Step`, `Test Data`, `Expected Result`, `Actual Result`, `Test Case Status`, `Automation Status`, `Notes`.
- API test case database has exact Notion display columns in this order: `TC ID`, `Scenario`, `Summary`, `Method`, `URL`, `Header`, `Params`, `Body`, `Expected Result`, `Actual Result`, `Test Case Status`, `Automation Status`, `Notes`.
- Default view, `All Cases`, and every qa-planner-created Notion view use the same visual property order; record `notion_view_column_order_gap` when a view cannot be updated and `notion_view_order_unverified` when order cannot be verified.
- If database/schema tools were unavailable, fallback Notion page contains a table with the same ordered display columns and records the database/schema gap.
- Notion page/database links were captured in `planning_state` and handoff contracts.
- Managed Notion artifacts avoid duplicate pages/databases for the same package id.
- Sensitive data is redacted before Notion write.

Status transition gates, apply only when Plane status movement is requested:
- Target Plane state was resolved with `list_states`.
- Workflow transitions follow `Todo Test -> Analyze Test`, `Analyze Test -> Ready to Test|Update Test`, `Update Test -> Need Review Test`, and `Need Review Test -> Ready to Test|Update Test`.
- Post-update status movement happens only after successful Plane sync.
- Target status not found skips transition without failing planning.

## Platform Fallback

Use platform-native document, spreadsheet, or JSON tools when available. If a platform lacks rendering support, produce Markdown and JSON artifacts from the field model. Core behavior must not depend on Codex-only, Claude-only, or Plane-only tools.

## Resources

Read only when needed:
- `references/plane-hybrid.md`: Plane MCP tool mapping, QA state workflow, parent/sub-work-item intake, sub-work-item CRUD and comments, attachment, planning enrichment, sync, wiki/page, links, idempotency, status movement, handoff tracking, and safety rules
- `references/notion-mcp.md`: Notion MCP tool mapping, test plan page output, mandatory test case database output, link capture, idempotency, batching, fallback, and safety rules
- `schemas/planning-state.schema.json`: canonical planning state schema
- `schemas/handoff-contract.schema.json`: downstream contract schema
- `schemas/review-feedback.schema.json`: OK/NOK/partial review feedback schema
- `schemas/notion-output-policy.schema.json`: Notion page/database write policy, only for Notion path
- `schemas/plane-source.schema.json`: Plane source schema, only for Plane path
- `schemas/plane-output-policy.schema.json`: Plane write and status policy schema, only for Plane path
- `schemas/plane-sync-record.schema.json`: Plane sync/idempotency schema, only for Plane path
- `templates/Template Test Case.xlsx`: spreadsheet baseline copied unchanged from source template
- `templates/test-plan.md`: human-readable test plan skeleton
- `templates/test-case.md`: human-readable test case skeleton
- `templates/test-case-api.md`: human-readable API test case skeleton
- `templates/handoff-contract.json`: multi-agent contract skeleton
- `templates/plane-comment.md`: Plane comment skeleton, only for Plane path
- `templates/plane-page.md`: Plane wiki/page skeleton, only for Plane path
- `examples/`: sample requirement, planning state, Plane input, and rendered outputs
