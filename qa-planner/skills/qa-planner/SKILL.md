---
name: qa-planner
description: Portable QA planning agent for converting requirements, PRDs, user stories, API specs, UI flows, screenshots, database notes, and technical constraints into governed planning_state JSON, test plans, test cases using the Template Test Case field model, coverage maps, OK/NOK review updates, and handoff contracts for qa-executor, qa-automation, and qa-reporter. Use when asked to create or revise QA plans, QA scenarios, test cases, requirement coverage, downstream QA handoffs, or planning review packages. Do not use for running tests, creating defect tickets, writing final automation scripts, or producing final execution reports.
---

# QA Planner

Use this skill to create and revise QA planning artifacts. Treat `planning_state.json` as the source of truth, then render human-readable plans, test cases, coverage maps, and handoff contracts from that state.

## Core Rule

Do planning work only. Do not execute tests, create defect tickets, produce final execution reports, write final automation scripts, invent selectors, invent credentials, or change execution statuses without verified executor results or explicit user instruction.

## Input Intake

Accept raw or structured inputs:
- requirements, PRDs, user stories, acceptance criteria, business rules, UI flows, screenshots, API specs, database notes, architecture notes, constraints, risks, and release scope
- existing test cases or QA plans that need normalization or revision
- human review feedback with `OK` or `NOK`

If critical information is missing, record it in `open_questions`. Ask one concise question only when the missing detail blocks useful planning.

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

## Platform Fallback

Use platform-native document, spreadsheet, or JSON tools when available. If a platform lacks rendering support, produce Markdown and JSON artifacts using the templates. Core behavior must not depend on Codex-only or Claude-only tools.

## Resources

Read only when needed:
- `schemas/planning-state.schema.json`: canonical planning state schema
- `schemas/handoff-contract.schema.json`: downstream contract schema
- `schemas/review-feedback.schema.json`: OK/NOK review feedback schema
- `templates/Template Test Case.xlsx`: spreadsheet baseline copied unchanged from source template
- `templates/test-plan.md`: human-readable test plan skeleton
- `templates/test-case.md`: human-readable test case skeleton
- `templates/handoff-contract.json`: multi-agent contract skeleton
- `examples/`: sample requirement, planning state, and rendered test cases
