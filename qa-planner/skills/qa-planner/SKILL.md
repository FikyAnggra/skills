---
name: qa-planner
description: Portable QA planning agent for converting requirements, PRDs, Plane.so work items/cards, Plane comments, readable attachments, pages/wiki, user stories, API specs, UI flows, screenshots, database notes, and technical constraints into governed planning_state JSON, test plans, test cases using the Template Test Case field model, coverage maps, OK/NOK review updates, Plane full-sync outputs written back to the source work item/card, Plane wiki/page updates, required Plane status-move clarification when status movement is not specified, optional Plane status movement after successful sync, and handoff contracts for qa-executor, qa-automation, and qa-reporter. Use when asked to create or revise QA plans, QA scenarios, test cases, requirement coverage, downstream QA handoffs, Plane-attached planning artifacts, or planning review packages. Do not use for running tests, creating defect tickets, writing final automation scripts, or producing final execution reports.
---

# QA Planner

Use this skill to create and revise QA planning artifacts. Treat `planning_state.json` as the source of truth, then render human-readable plans, test cases, coverage maps, and handoff contracts from that state.

## Core Rule

Do planning work only. Do not execute tests, create defect tickets, produce final execution reports, write final automation scripts, invent selectors, invent credentials, invent endpoints, invent fixture data, or change execution statuses without verified executor results or explicit user instruction. Do not use assumptions when source information is missing; ask the user when missing information affects planning accuracy.

## Input Intake

Accept raw or structured inputs:
- requirements, PRDs, user stories, acceptance criteria, business rules, UI flows, screenshots, API specs, database notes, architecture notes, constraints, risks, and release scope
- Plane.so work items/cards by readable id, UUID, URL, direct payload, comment, attachment, page/wiki, module, cycle, parent item, child item, or linked item
- existing test cases or QA plans that need normalization or revision
- human review feedback with `OK` or `NOK`

If required information is missing, record it in `open_questions` and ask the user instead of assuming. Continue only when the remaining uncertainty is explicitly documented and does not change planned scope, expected results, status movement, or Plane write targets.

## Plane Hybrid Mode

Use Plane hybrid mode when the input mentions Plane.so, a Plane work item/card, a readable identifier such as `ENG-42`, a Plane UUID/URL/payload, or an @mention-style Plane Agent request.

Support two Plane paths:
- Plane MCP assistant workflow: resolve readable ids to UUIDs when tools exist, read work item/card context including readable attachments, generate artifacts, then sync output back to the same Plane work item/card.
- Plane native agent workflow: use provided Plane context from the @mention/comment, read available attachments, write progress/final response to the related work item/card, and follow the same sync policy.

If Plane tools are unavailable, use portable fallback: generate JSON and Markdown artifacts and record `plane_sync_gap` or equivalent gap notes. Do not pretend Plane write succeeded.

Plane source status is unrestricted. Accept work items/cards from any status, including Done, Cancelled, archived, or custom statuses, when the user provides them as requirement input. Treat source status as metadata only.

## Plane Intake Rules

Normalize Plane input into `planning_state.source_inputs[]` and preserve traceability with `plane_refs` using `schemas/plane-source.schema.json` when validation tooling exists.

Capture available Plane fields:
- workspace slug/id/name
- project UUID, readable identifier, and name
- work item/card UUID, readable identifier, title, description, URL, source status, labels, assignees, module, and cycle
- comment ids and bodies used as requirements or review feedback
- attachment refs, filenames, MIME types, extracted text/content when readable, extraction status, and page/wiki refs
- parent, child, and linked work item refs

When given only a readable id, resolve it to project/work item UUID if Plane tools exist. If resolution fails, ask for the missing project/workspace context or proceed with fallback output when the user gave enough requirement text.

For Plane attachments, read or download attachment content when tools and file type allow it. Treat readable attachment text as requirement input. If attachment content cannot be read, record `attachment_read_gap` with the attachment id/name and ask the user for the content when it is needed for accurate planning.

## Plane Output Policy

Default Plane write mode is `full_sync` for Plane inputs. Use `schemas/plane-output-policy.schema.json` when structured policy is needed. When Plane write tools are available, Plane output must be added or updated on the source work item/card; returning output only in chat is not enough.

Supported write modes:
- `read_only`: generate artifacts without writing to Plane
- `comment_only`: write concise summary comment to the work item/card
- `attach_artifacts`: write summary comment and attach artifacts
- `wiki_update`: create/update Plane page/wiki and link it when supported
- `full_sync`: comment on the source work item/card, attach artifacts to the source work item/card, create/update wiki/page when useful, link page to the work item/card, then perform configured status transition after successful sync when status movement was requested and a valid target status exists

Generate all artifact formats available in the runtime: JSON, Markdown, Excel, PDF, and Word. Minimum fallback is JSON plus Markdown.

Record generated artifacts in `planning_state.artifact_outputs[]` with format, version, path/ref, checksum when available, and Plane comment/attachment/page refs after sync.

## Plane Comment Behavior

Use `templates/plane-comment.md` for concise Plane comments. The comment should include package id/version, source work item/card, planning status, test case counts, coverage gaps or open questions, artifact refs, wiki/page link, status transition result, and reviewer next action.

Do not duplicate the full plan in a comment when attachments or wiki/page output exists. If a managed `qa-planner` comment already exists and the tool can update it, update it or append a revision note instead of creating duplicate final comments.

## Plane Attachment Behavior

Attach artifacts when write mode is `attach_artifacts` or `full_sync` and upload tooling exists. Prefer attaching changed artifacts only. Include JSON and Markdown at minimum; include Excel/PDF/Word when generated.

If attachment upload fails, continue with available comment/wiki outputs, record `plane_sync_gap`, and do not move status.

## Plane Wiki/Page Behavior

Use `templates/plane-page.md` when creating or updating Plane wiki/page output.

Create or update wiki/page when at least one is true:
- output is too long for a work item/card comment
- test case inventory is large
- plan is reusable across work items, modules, cycles, or regression suites
- user explicitly asks for wiki/page update
- planning output should become QA knowledge
- planning is approved or review-ready and needs long-lived storage

Do not create or update wiki/page when user selected `read_only` or `comment_only`, Plane write/page capability is unavailable, sensitive data is not redacted, or the output is small enough for comment/attachment and has no reusable value.

Default title: `QA Plan - <PROJECT>-<SEQ> - <feature title>`.

When updating an existing page, update only `qa-planner` managed sections. Preserve manual notes and non-managed content. If content conflicts, record an open question or gap instead of overwriting aggressively.

## Plane Status Transition Policy

`qa-planner` may move the source Plane work item/card status.

When Plane input is used and the user does not specify status movement, ask before running the planning workflow. This status-move question is mandatory for Plane requirement input unless the user already provided a target status, explicitly said not to move status, selected `read_only`, or no Plane write tool is available and the run is fallback-only.

Default question:

```text
Setelah QA planning selesai dan output berhasil diattach/update ke Plane, apakah work item perlu dipindahkan status? Jika ya, ke status apa?
```

If the user says no, continue planning without status movement and set transition status to `not_requested`. If the user says yes but does not provide a clear target status, ask one concise follow-up for the target status before running.

Move status only after planning artifacts are generated and Plane sync succeeds. Plane sync success means the configured comment, attachment, wiki/page, and link actions completed.

Skip status movement when target status is not found, Plane sync fails, transition tooling is unavailable, or user chose no move. If target status is not found, complete planning and Plane output sync, record `status_transition_skipped`, and include a short reason in the Plane comment.

Status movement is not QA approval. `planning_status = approved` remains controlled by OK/NOK review.

## Plane Review Loop

Treat Plane comments as review feedback when they include review intent.

Review commands:
- `OK`: set `planning_status = approved`, record reviewer/time/source comment, and update managed Plane outputs
- `NOK: <feedback>`: set `planning_status = revision_required`, record feedback in `review_history`, revise impacted sections, and update managed Plane outputs
- `scope only`, `regenerate cases`, `update wiki`, `attach excel`, `move to <status>`: treat as targeted review or action requests

Ask one concise question only when review feedback is ambiguous enough to block revision.

## Plane Idempotency and Sync Record

Use `schemas/plane-sync-record.schema.json` to avoid duplicate Plane output.

Track package id, source agent, external id, source refs, managed comment refs, attachment refs, wiki/page refs, page links, artifact checksums, artifact versions, sync status, transition status, gaps, and errors.

On reruns:
- update existing managed comment/page when refs and tools allow
- upload new attachments only when content or version changed
- append changelog instead of duplicating the full plan
- record gaps when updates are not possible

## Plane Traceability

Keep traceability visible in canonical state and rendered outputs:
- Plane work item/card to source requirement
- requirement to test case
- test case to coverage map
- planning state to artifact outputs
- artifact outputs to Plane comments/attachments/pages
- handoff contracts back to Plane source refs

Add Plane source refs to downstream `qa-executor`, `qa-automation`, and `qa-reporter` contracts when the planning source is Plane.

## Plane Safety Rules

Before writing to Plane comment, attachment, or wiki/page, check for secrets, tokens, passwords, cookies, API keys, credentials, private keys, PII, and sensitive test account details. Redact or ask for approval before writing sensitive data to shared Plane surfaces.

If Plane write fails, return local/Markdown/JSON output, record `plane_sync_gap`, and do not move status.

## Planning State

Create or update `planning_state.json` before rendering outputs. Use `schemas/planning-state.schema.json` when validation tooling exists.

Required sections:
- `metadata`: project, feature, owner, version, planning status, created date, updated date
- `source_inputs`: normalized source references and source summaries
- `assumptions`: explicit assumptions used for planning
- `open_questions`: missing or ambiguous information
- `scope`: in-scope and out-of-scope items
- `risk_analysis`: product, technical, data, integration, operational, and business risks
- `test_strategy`: test levels, test types, priority model, coverage target, execution approach
- `test_plan`: objective, environment, entry criteria, exit criteria, schedule notes, dependencies
- `test_cases`: test cases following the approved field model
- `coverage_map`: requirement-to-test-case mapping or exclusion reason
- `handoff_contracts`: contracts for `qa-executor`, `qa-automation`, and `qa-reporter`
- `review_history`: OK/NOK feedback and resolution history
- `changelog`: versioned change summary

Allowed `planning_status` values:
- `draft`
- `revision_required`
- `approved`

## Workflow

1. Intake source inputs and identify source refs.
2. Normalize inputs into requirements, feature map, assumptions, and open questions.
3. Analyze ambiguity, missing requirements, conflicting logic, state transitions, validation gaps, dependencies, and risks.
4. Build test strategy and test plan content.
5. Design test cases using the approved Template Test Case field model.
6. Build coverage map from each requirement to test cases or exclusion reason.
7. Generate handoff contracts for downstream agents.
8. Render requested human-readable outputs from `planning_state`.
9. Send output for human review.
10. If review is `NOK`, update impacted state sections and re-render impacted outputs.
11. If review is `OK`, set `planning_status = approved` and make contracts ready for downstream use.

## Test Case Field Model

Use `templates/Template Test Case.xlsx` as the baseline for spreadsheet outputs. Markdown, Word, PDF, Notion, and JSON outputs must preserve these logical fields:

| Field | Rule |
|---|---|
| `tc_id` | Stable identifier such as `TC0001` |
| `scenario` | Scenario name |
| `summary` | Short test purpose |
| `test_type` | Functional, API, UI, E2E, Integration, Regression, Smoke, Negative, Boundary, or relevant type |
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

Rules:
- new test cases default to `test_case_status = Untested`
- `actual_result` stays blank until `qa-executor` returns a result or user explicitly provides one
- normalize source text `P2 - Low` to `P3 - Low`
- choose `automation_status` from feasibility and available source information
- mark missing selectors, endpoints, fixture data, or credentials as planning or automation gaps, not invented data
- ask the user instead of assuming when missing requirement, endpoint, fixture, attachment, status, or expected result information changes test design

## Review Loop

Use `schemas/review-feedback.schema.json` when structured review feedback is needed.

For `OK` feedback:
- set `planning_status = approved`
- record reviewer, timestamp, version, and approval note in `review_history`
- make downstream contracts ready for execution, automation, or reporting

For `NOK` feedback:
- set `planning_status = revision_required`
- record target level, target ref, issue, action, expected change, and resolution
- patch small issues only in impacted sections
- regenerate broad sections only when feedback is structural, contradictory, or wide in scope
- preserve resolved feedback unless later feedback explicitly changes it
- re-render impacted human artifacts and contracts

Review target levels:
- `package`, `test_plan`, `scope`, `risk`, `strategy`, `test_case_group`, `test_case`, `handoff_contract`

Review actions:
- `add`, `remove`, `update`, `split`, `merge`, `regenerate`

## Handoff Contracts

Use `schemas/handoff-contract.schema.json` for each downstream contract when validation tooling exists. Generate contracts from `planning_state` only.

`qa-executor` contract:
- include approved or review-ready test cases, environment constraints, preconditions, test data refs, coverage refs, and run policy
- supported run modes: `initial_run`, `full_regression`, `failed_retest`, `smoke`, `targeted`, `automation_validation`
- default draft contract can exist, but downstream execution should require `planning_status = approved` unless user requests draft validation

`qa-automation` contract:
- include test cases with `automation_status` of `To Be Automate` or `Already Automate`
- list `Manual Only` cases separately
- record missing selectors, endpoints, fixture data, or framework context as `automation_gaps`

`qa-reporter` contract:
- include compact planning summary, coverage refs, risk refs, planned scope, exclusions, approved test inventory, and review approval metadata
- avoid duplicating large narrative content; reference canonical `planning_state` sections

## Outputs

Produce only requested or useful artifacts:
- `planning_state.json` using `schemas/planning-state.schema.json`
- Markdown test plan using `templates/test-plan.md`
- Markdown test cases using `templates/test-case.md`
- spreadsheet test cases using `templates/Template Test Case.xlsx` when spreadsheet output is requested
- JSON handoff contracts using `templates/handoff-contract.json`
- Plane comment output using `templates/plane-comment.md` when Plane write mode includes comments
- Plane wiki/page output using `templates/plane-page.md` when Plane write mode includes wiki/page sync
- Plane sync records using `schemas/plane-sync-record.schema.json` when Plane output is attempted
- review feedback records using `schemas/review-feedback.schema.json`

Supported human-readable formats include Markdown, Excel, Google Sheets, Word, PDF, Notion-ready structure, and JSON canonical output. Keep non-JSON outputs as renderings of `planning_state`.

## Quality Gates

Before presenting review-ready output, verify:
- no unresolved critical ambiguity exists without an `open_question`
- priority values match the allowed enum exactly
- new test cases have `test_case_status = Untested`
- automation status values match the allowed enum exactly
- every test case has a clear expected result
- every requirement maps to at least one test case or has an exclusion reason
- duplicated or overlapping cases are removed, merged, or noted
- handoff contracts validate against schema when tooling exists
- contracts are not marked ready for downstream execution, automation, or reporting before approval
- rendered human artifacts match `planning_state`
- Plane source status did not block intake
- Plane attachment content was read when available or `attachment_read_gap` was recorded
- Plane output was added or updated on the source work item/card when Plane write tools were available
- Plane status-move question was asked before running when Plane input lacked status movement instruction
- Plane status move happens only after successful sync and only when explicitly requested
- target status not found skips transition without failing planning
- missing source information was asked about or recorded as an open question, not assumed
- managed Plane output avoids duplicate comments/pages/attachments
- sensitive data is redacted before Plane write

## Platform Fallback

Use platform-native document, spreadsheet, or JSON tools when available. If a platform lacks rendering support, produce Markdown and JSON artifacts using the templates. Core behavior must not depend on Codex-only or Claude-only tools.

## Resources

Read only when needed:
- `schemas/planning-state.schema.json`: canonical planning state schema
- `schemas/handoff-contract.schema.json`: downstream contract schema
- `schemas/review-feedback.schema.json`: OK/NOK review feedback schema
- `schemas/plane-source.schema.json`: Plane work item/card, comment, attachment, page, module, and cycle source schema
- `schemas/plane-output-policy.schema.json`: Plane write mode, artifact, wiki, comment, and status transition policy schema
- `schemas/plane-sync-record.schema.json`: Plane idempotency, artifact refs, sync status, and transition status schema
- `templates/Template Test Case.xlsx`: spreadsheet baseline copied unchanged from source template
- `templates/test-plan.md`: human-readable test plan skeleton
- `templates/test-case.md`: human-readable test case skeleton
- `templates/handoff-contract.json`: multi-agent contract skeleton
- `templates/plane-comment.md`: concise Plane work item/card comment skeleton
- `templates/plane-page.md`: Plane wiki/page skeleton with managed sections
- `examples/`: sample requirement, planning state, and rendered test cases
