# QA-Planner Portable Agent Skill Design

Date: 2026-06-10
Status: Approved design draft for user review
Source artifacts:
- `QA Agent & Workflow.pdf`
- `Template Test Case.xlsx`

## 1. Purpose

`qa-planner` is a portable QA planning agent skill/package. It creates and revises QA planning artifacts until human review returns `OK`.

The agent supports a dual-track output model:
- Human-readable QA artifacts: test plan and test cases in Markdown, Google Docs, Google Sheets, Notion, Excel, Word, PDF, or another supported document format.
- Machine-readable handoff contracts: canonical JSON contracts for downstream QA agents.

The skill must be platform-agnostic. It should work as a portable skill package, such as a folder installable through a skills manager like `skills.sh` or `npx skills add ...`, and must not depend on Codex-only behavior for its core logic.

## 2. Scope

`qa-planner` owns planning and revision work only.

In scope:
- Normalize raw requirements, PRDs, user stories, API specs, UI flows, screenshots, database notes, and technical constraints.
- Identify ambiguity, missing requirements, conflicting logic, state transitions, validation gaps, risk areas, and dependencies.
- Generate a canonical `planning_state`.
- Generate test plan content.
- Generate test cases using the `Template Test Case.xlsx` field model.
- Generate requirement coverage mapping.
- Generate handoff contracts for `qa-executor`, `qa-automation`, and `qa-reporter`.
- Support `OK` / `NOK` human review loops.
- Revise artifacts until reviewed output is correct.

Out of scope:
- Running tests.
- Creating defect tickets.
- Producing final test execution reports.
- Writing final automation scripts.
- Updating execution result status without an executor result.
- Inventing selectors, API endpoints, credentials, or test data without a source.

## 3. Source of Truth

`planning_state` is the canonical source of truth.

Human-readable outputs and machine handoff contracts are rendered from `planning_state`. This prevents duplicated context, keeps revisions controlled, and reduces token/context cost during repeated review loops.

Recommended lifecycle:
1. Create or update `planning_state`.
2. Render human-readable output from `planning_state`.
3. Render handoff contracts from `planning_state`.
4. Human reviews output.
5. If review result is `NOK`, update `planning_state` based on feedback.
6. Re-render impacted outputs.
7. Repeat until review result is `OK`.

## 4. Planning State Model

`planning_state` should contain these sections:

- `metadata`: project, feature, owner, version, status, created date, updated date.
- `source_inputs`: requirement, PRD, API spec, UI flow, screenshot, database notes, constraints.
- `assumptions`: assumptions made by the planner.
- `open_questions`: ambiguity or missing information requiring human input.
- `scope`: in-scope and out-of-scope items.
- `risk_analysis`: product, technical, data, integration, operational, and business risks.
- `test_strategy`: test levels, test types, coverage target, priority model, execution approach.
- `test_plan`: objective, environment, entry criteria, exit criteria, schedule notes, dependencies.
- `test_cases`: test case records following the approved template field model.
- `coverage_map`: requirement-to-test-case mapping.
- `handoff_contracts`: downstream contracts for executor, automation, and reporter.
- `review_history`: human review status, feedback, target area, action, and resolution.
- `changelog`: versioned change summary.

## 5. Test Case Template Model

`Template Test Case.xlsx` is the baseline for all test case outputs. Excel and Google Sheets should preserve this structure. Markdown, Notion, Word, PDF, and other formats should render the same fields in a format-native layout.

Required test case fields:

| Field | Description |
|---|---|
| `tc_id` | Test case identifier, for example `TC0001` |
| `scenario` | Scenario name |
| `summary` | Short purpose of the test |
| `test_type` | Functional, API, UI, E2E, Integration, Regression, Smoke, Negative, Boundary, or other relevant type |
| `priority` | `P0 - Critical`, `P1 - High`, `P2 - Medium`, or `P3 - Low` |
| `pre_conditions` | Preconditions before execution |
| `test_steps` | Ordered test steps |
| `test_data` | Input data, fixture, account, payload, or dataset |
| `expected_result` | Expected observable result |
| `actual_result` | Execution result, blank before execution |
| `test_case_status` | Latest execution status |
| `automation_status` | Automation treatment |
| `notes` | Additional planner notes, gaps, or constraints |

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

Default values:
- New test cases default to `test_case_status = Untested`.
- `actual_result` remains blank until `qa-executor` returns an execution result.
- `automation_status` must be selected based on feasibility and available source information.

Template normalization:
- If any source/template text contains `P2 - Low`, normalize it to `P3 - Low`.

## 6. Status Model

Planning approval and execution result are separate concerns.

`planning_status` values:
- `draft`
- `revision_required`
- `approved`

`test_case_status` values:
- `Untested`
- `On Progress`
- `Passed`
- `Failed`
- `Blocked`
- `Retest`

`automation_status` values:
- `Manual Only`
- `To Be Automate`
- `Already Automate`

Rules:
- `planning_status = approved` means the planning package is valid and may be used downstream.
- `test_case_status` reflects latest execution state, not planning approval.
- `qa-planner` must not change `test_case_status` to `Passed`, `Failed`, `Blocked`, or `Retest` unless the value comes from a verified executor result or explicit user instruction.

## 7. Review and Revision Loop

Human review result is either `OK` or `NOK`.

If result is `OK`:
- Set `planning_status = approved`.
- Mark package as approved version.
- Handoff contracts become ready for downstream agents.

If result is `NOK`:
- Capture feedback in structured form.
- Identify target area.
- Apply the required change.
- Re-render impacted human artifacts and contracts.
- Continue until feedback is resolved.

Review feedback fields:

| Field | Description |
|---|---|
| `review_status` | `OK` or `NOK` |
| `target_level` | Package, plan, scope, risk, strategy, test case group, test case, or handoff contract |
| `target_ref` | Section id, group id, test case id, or contract path |
| `issue` | Human review issue |
| `action` | `add`, `remove`, `update`, `split`, `merge`, or `regenerate` |
| `expected_change` | Expected correction |
| `resolution` | What the planner changed |

Allowed `target_level` values:
- `package`
- `test_plan`
- `scope`
- `risk`
- `strategy`
- `test_case_group`
- `test_case`
- `handoff_contract`

Allowed `action` values:
- `add`
- `remove`
- `update`
- `split`
- `merge`
- `regenerate`

Revision strategy:
- Patch impacted sections for small feedback.
- Regenerate section/package only when feedback is broad, structural, or contradictory.
- Always preserve resolved feedback unless later feedback explicitly changes it.

## 8. Handoff Contracts

Handoff contracts are generated only from approved or review-ready `planning_state`. Downstream execution should require `planning_status = approved` unless a human explicitly requests dry-run or draft validation.

### 8.1 `qa-executor` Contract

Purpose: tell `qa-executor` what can be executed and what should be selected for a specific run.

Executor data sources:
- Approved test cases from `planning_state.test_cases`.
- Test plan environment and execution constraints.
- Test data and preconditions.
- Requirement coverage map.
- Run policy.

`approved` does not mean every test case must run every time. It means the test case is valid for execution. Actual execution is controlled by `run_policy`.

Supported run modes:
- `initial_run`: approved test cases with `Untested` status.
- `full_regression`: all approved test cases.
- `failed_retest`: test cases with `Failed` or `Retest` status.
- `smoke`: priority/tagged smoke subset.
- `targeted`: explicitly selected test case ids.
- `automation_validation`: test cases related to automation candidates.

Example executor contract:

```json
{
  "contract_version": "1.0",
  "source_agent": "qa-planner",
  "target_agent": "qa-executor",
  "package_id": "QA-PLAN-AUTH-LOGIN-v0.2",
  "approval_status": "approved",
  "run_policy": {
    "run_mode": "initial_run",
    "status_filter": ["Untested"],
    "priority_filter": ["P0 - Critical", "P1 - High", "P2 - Medium", "P3 - Low"],
    "include_tags": [],
    "exclude_tags": []
  },
  "execution_scope": {
    "candidate_test_cases": ["TC0001", "TC0002", "TC0003"],
    "selected_test_cases": ["TC0001", "TC0002", "TC0003"]
  }
}
```

### 8.2 `qa-automation` Contract

Purpose: tell `qa-automation` which approved test cases should become or update automation scripts.

Automation data sources:
- Approved test cases.
- `automation_status`.
- Test steps, test data, expected result.
- Automation hints when available.
- Repository/framework context when available.

Automation rules:
- `To Be Automate`: create new automation script.
- `Already Automate`: validate/update existing automation script.
- `Manual Only`: do not automate unless human changes the status.
- Missing selectors, endpoints, or fixture data must be recorded as `automation_gap`, not invented.

Example automation contract:

```json
{
  "contract_version": "1.0",
  "source_agent": "qa-planner",
  "target_agent": "qa-automation",
  "package_id": "QA-PLAN-AUTH-LOGIN-v0.2",
  "approval_status": "approved",
  "automation_policy": {
    "include_status": ["To Be Automate", "Already Automate"],
    "exclude_status": ["Manual Only"],
    "priority_filter": ["P0 - Critical", "P1 - High"]
  },
  "automation_scope": {
    "create_scripts": ["TC0001"],
    "update_scripts": ["TC0005"],
    "manual_only": ["TC0002"]
  }
}
```

### 8.3 `qa-reporter` Contract

Purpose: provide planning context for SIT/UAT/report generation.

Reporter data sources:
- Requirement coverage map.
- Planned scope and excluded scope.
- Risk analysis.
- Approved test case inventory.
- Review approval metadata.
- Execution result references after executor runs.

The reporter contract should avoid duplicating large narrative sections. It should reference canonical sections and include compact summary fields.

## 9. Workflow

1. Intake source inputs.
2. Normalize inputs into requirement list, feature map, assumptions, and open questions.
3. Analyze ambiguity, gaps, conflicts, edge cases, state transitions, validation gaps, risks, and dependencies.
4. Build test plan content.
5. Design test cases using the approved template model.
6. Build coverage map.
7. Generate handoff contracts.
8. Render human-readable output in requested format.
9. Send for human review.
10. If `NOK`, revise based on feedback.
11. If `OK`, mark `planning_status = approved`.

## 10. Quality Gates

Before sending output for review, `qa-planner` must check:
- No unresolved critical ambiguity without an explicit open question.
- Priority uses only `P0 - Critical`, `P1 - High`, `P2 - Medium`, `P3 - Low`.
- New test cases default to `Untested`.
- Automation status uses only allowed values.
- Each test case has a clear expected result.
- Each requirement maps to at least one test case or has an exclusion reason.
- Duplicated or overlapping test cases are removed, merged, or noted.
- Handoff contracts validate against their schema.
- Contracts are not marked ready for execution/automation/reporting before approval.

## 11. Portable Skill Package Structure

Target package:

```text
qa-planner/
  .codex-plugin/
    plugin.json
  .claude-plugin/
    plugin.json
  skills/
    qa-planner/
      SKILL.md
      schemas/
        planning-state.schema.json
        handoff-contract.schema.json
        review-feedback.schema.json
      templates/
        Template Test Case.xlsx
        test-plan.md
        test-case.md
        handoff-contract.json
      examples/
        input-requirement.md
        planning-state.example.json
        output-test-cases.example.md
      agents/
        openai.yaml
  README.md
  CHANGELOG.md
```

Core behavior must be platform-neutral. Codex and Claude plugin wrappers are runtime packaging layers, not required for agent behavior. `CHANGELOG.md` records versioned changes, compatibility updates, schema/template changes, and migration notes.

## 12. Token and Context Strategy

To control token usage:
- Treat `planning_state` as canonical state.
- Store review feedback as deltas in `review_history`.
- Load only impacted sections during revision when possible.
- Render large human documents from state instead of keeping full copies in prompt context.
- Keep handoff contracts compact and reference canonical sections.
- Store version history as changelog summaries, not full duplicated artifacts.

## 13. Implementation Notes

The implementation phase should produce:
- portable `qa-planner` skill folder
- Codex plugin wrapper
- Claude plugin wrapper
- JSON schemas for planning state, handoff contract, and review feedback
- Template mapping from `Template Test Case.xlsx` to canonical test case fields
- Example input and output artifacts
- `README.md` and `CHANGELOG.md`
- Validation checklist for generated outputs

No runtime or platform-specific dependency should be required for the core skill instructions.
