---
name: qa-planner
description: Portable QA planning agent for converting requirements, PRDs, user stories, API specs, UI flows, screenshots, database notes, technical constraints, optional Plane.so work items/cards, and optional Notion pages/databases into governed planning_state JSON, test plans, test cases using the Template Test Case field model, coverage maps, OK/NOK or partial review updates, Notion test plan pages, Notion test case databases, copied Notion links, and handoff contracts for qa-executor, qa-automation, and qa-reporter. Use when asked to create or revise QA plans, QA scenarios, test cases, requirement coverage, downstream QA handoffs, Plane-attached planning artifacts, Notion-published planning artifacts, or planning review packages. Do not use for running tests, creating defect tickets, writing final automation scripts, or producing final execution reports.
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
- Plane.so work items/cards, comments, readable attachments, pages/wiki, modules, cycles, parent/child items, or linked items
- Notion pages/databases, Notion output destinations, or requests to publish/copy/share Notion links for test plans and test cases

If the input contains Plane.so signals such as `Plane`, a Plane work item/card, a readable id like `ENG-42`, a Plane URL/UUID/payload, or an @mention-style Plane Agent request, read `references/plane-hybrid.md` before continuing.

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
- `test_cases`: test cases following the approved field model
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
  "test_cases": [],
  "coverage_map": [],
  "handoff_contracts": { "qa_executor": {}, "qa_automation": {}, "qa_reporter": {} },
  "review_history": [],
  "changelog": []
}
```

## Workflow

Choose one path first.

Standard path:
1. Intake source inputs and identify source refs.
2. Normalize inputs into requirements, feature map, assumptions, and open questions.
3. Classify ambiguity as blocking or non-blocking using the rules above; ask only for blocking ambiguity and record non-blocking ambiguity in `open_questions`.
4. Analyze ambiguity, gaps, conflicts, edge cases, state transitions, validation gaps, risks, and dependencies.
5. Build test strategy and test plan content.
6. Design test cases using the approved Template Test Case field model.
7. Build coverage map.
8. Render artifacts using the output decision rules.
9. Apply review feedback until the package is approved or remaining gaps are explicit.

Plane path:
1. Read `references/plane-hybrid.md`.
2. Resolve Plane source refs and read the work item/card plus readable attachments when tools allow.
3. Ask the mandatory status-move question before running when Plane input lacks status movement instruction.
4. Continue with the standard path.
5. Sync outputs back to the source Plane work item/card according to the Plane output policy.
6. Move Plane status only after successful sync and only when the user requested a valid target status.

Notion path:
1. Read `references/notion-mcp.md`.
2. Resolve destination parent page/database or ask the user when no destination is clear.
3. Continue with the standard path.
4. Create or update a Notion test plan page.
5. Create or update a Notion test case database. Test cases must use a Notion database with the exact Template Test Case columns when `notion-create-database` and required `notion-update-data-source` support are available.
6. Store Notion page/database URLs in `planning_state.artifact_outputs`, `planning_state.notion_context`, and handoff contracts.

Large-input handling:
- If estimated test cases exceed 50, propose batching by feature, module, epic, API group, or risk priority before generating all cases.
- If estimated test cases exceed 100 or source input is too large to hold safely, ask the user to choose first batch scope.
- Preserve one `planning_state` and append batches into `test_cases`, `coverage_map`, `review_history`, and `changelog`.
- Never truncate silently. Record omitted scope explicitly.

## Test Case Field Model

Use `templates/Template Test Case.xlsx` as the spreadsheet baseline. Markdown, Word, PDF, Notion, and JSON outputs must preserve these logical fields:

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
- `Already Automate`: choose only when source references an existing script, suite, test id, CI job, or maintained automation coverage.
- `To Be Automate`: choose for repetitive regression candidates with stable UI/API behavior, deterministic data, observable expected result, and known or discoverable automation surface.
- `Manual Only`: choose for exploratory testing, subjective visual judgment, usability review, one-off investigation, unsupported external dependency, unstable selectors, missing fixtures, CAPTCHA/OTP/manual approval, or high automation cost compared with regression value.
- Borderline cases default to `Manual Only` plus a note explaining what information would make automation feasible.

Rules:
- New test cases default to `test_case_status = Untested`.
- `actual_result` stays blank until `qa-executor` returns a result or user explicitly provides one.
- If input priority is not one of the allowed values, add an `open_question` and ask the user to clarify. Do not downgrade or remap invalid priority silently.
- Apply the automation status decision rules above; when evidence is insufficient, use `Manual Only` and record the gap.
- Mark missing selectors, endpoints, fixture data, or credentials as gaps, not invented data.

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

Produce additional artifacts only under these rules:
- Test plan: produce for new planning packages or when plan sections changed.
- Test cases: produce when requirements, scope, risks, or cases changed.
- Coverage map: produce when requirements or test cases changed.
- Handoff contracts: produce only when approved, explicitly requested, or clearly marked draft.
- Spreadsheet: produce when user asks for Excel/Sheets, when Plane/full-sync policy requests attachments, or when test cases are large enough that a table is hard to review in Markdown.
- Plane comment/page/sync outputs: produce only when Plane path is active and Plane write policy allows it.
- Notion page/database outputs: produce only when Notion path is active and Notion write policy allows it. Test plan output is a page rendered from `templates/test-plan.md`; test case output is a database with exact `templates/test-case.md` columns unless database/schema tooling is unavailable, in which case create a Notion page containing that same test case table.

If schemas or templates are unavailable, generate inline JSON/Markdown structures from the field model and rules in this skill. Do not fail solely because `/schemas/` or `/templates/` files are unavailable.

## Quality Gates

Content gates, always apply:
- No unresolved critical missing or conflicting source information without an `open_question`.
- Priority values match the allowed enum exactly.
- New test cases have `test_case_status = Untested`.
- Automation status values match the allowed enum exactly.
- Each test case has a clear expected result.
- Each requirement maps to at least one test case or has an exclusion reason.
- Duplicated or overlapping cases are removed, merged, or noted.
- Rendered human artifacts match `planning_state`.

Plane sync gates, apply only on Plane path:
- Plane source status did not block intake.
- Plane MCP context was resolved when tools were available: work item payload, comments, activities, relations, states, members, properties, types, and requested cycle/module scope.
- Readable attachment content was read or `attachment_read_gap` was recorded.
- Plane output was added or updated on the source work item/card when write tools were available.
- Managed Plane output avoids duplicate comments/pages/attachments.
- Plane handoff tracker work items are created only when requested or required by an approved handoff contract.
- Sensitive data is redacted before Plane write.

Notion sync gates, apply only on Notion path:
- Destination parent page/database was resolved or the user approved fallback.
- Test plan was created or updated as a Notion page when write tools were available.
- Test plan page follows the structure of `templates/test-plan.md` and avoids raw JSON dumps.
- Test cases were created or updated as a Notion database when `notion-create-database` and required `notion-update-data-source` support were available.
- Test case database has exact Notion display columns in this order: `TC ID`, `Scenario`, `Summary`, `Test Type`, `Priority`, `Pre Conditions`, `Test Steps`, `Test Data`, `Expected Result`, `Actual Result`, `Test Case Status`, `Automation Status`, `Notes`.
- If database/schema tools were unavailable, fallback Notion page contains a table with the same ordered display columns and records the database/schema gap.
- Notion page/database links were captured in `planning_state` and handoff contracts.
- Managed Notion artifacts avoid duplicate pages/databases for the same package id.
- Sensitive data is redacted before Notion write.

Status transition gates, apply only when Plane status movement is requested:
- Status-move question was asked before running when Plane input lacked movement instruction.
- Status movement happens only after successful Plane sync.
- Target status not found skips transition without failing planning.

## Platform Fallback

Use platform-native document, spreadsheet, or JSON tools when available. If a platform lacks rendering support, produce Markdown and JSON artifacts from the field model. Core behavior must not depend on Codex-only, Claude-only, or Plane-only tools.

## Resources

Read only when needed:
- `references/plane-hybrid.md`: Plane MCP tool mapping, intake, attachment, planning enrichment, sync, wiki/page, links, idempotency, status movement, handoff tracking, and safety rules
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
- `templates/handoff-contract.json`: multi-agent contract skeleton
- `templates/plane-comment.md`: Plane comment skeleton, only for Plane path
- `templates/plane-page.md`: Plane wiki/page skeleton, only for Plane path
- `examples/`: sample requirement, planning state, Plane input, and rendered outputs
