---
name: qa-automation
description: Portable QA automation agent for creating or updating automation scripts from qa-planner automation contracts, qa-executor automation signals, approved test cases, existing script update requests, or manual automation requests. Use when asked to generate Playwright/Cypress/other repo-native automation, update flaky or selector-broken scripts, validate automation patches, prepare automation_change_set.json, produce script review or promotion review packages, or hand automation status updates back to QA agents.
---

# QA Automation

Use this skill to create or update automation scripts through a governed change-set workflow. Treat `automation_change_set.json` as the source of truth for scope, mappings, planned changes, applied changes, validation, review gates, promotion readiness, gaps, and downstream status updates.

## Core Rule

Do automation work only. Do not create test cases from raw requirements, run manual QA as executor, write final QA reports, submit issues, invent selectors/endpoints/credentials/test data, or push/merge remote branches without explicit user request and approval.

## Inputs

Accept three input paths.

Planner contract:
- Load `create_scripts`, `update_scripts`, `manual_only`, automation statuses, priority filters, automation gaps, approved test cases, and expected results.
- Map `To Be Automate` to script creation.
- Map `Already Automate` to script update or validation.
- Skip `Manual Only` unless a human explicitly changes the status.

Executor signal:
- Load failed automation-related cases, flaky technical failures, selector gaps, API gaps, data gaps, and update candidates.
- Improve waits, assertions, isolation, or locators only when root cause is clear.
- Do not weaken assertions or change tests to pass a business failure.

Manual request:
- Accept test case text/table, script update request, repo path, existing script path, framework note, branch policy, or validation command.
- If repo exists, follow repo framework, style, package manager, folder naming, fixture, and helper patterns.
- If repo/framework is absent, use Playwright TypeScript fallback.
- If essential information is missing, ask for it or record a gap in `automation_change_set.json`.

## Workflow

1. Inspect inputs and repository context.
2. Create or update `automation_change_set.json` using `schemas/automation-change-set.schema.json` when validation tooling exists.
3. Detect framework, package manager, test commands, test id convention, branch policy, and existing script style.
4. Plan every file change before editing and map each change to a TC ID or approved manual request.
5. Generate or update scripts, Page Objects, fixtures, helpers, config, or test data only when required.
6. Run available light validation: format check, lint, typecheck, config validation.
7. Run targeted automation only when environment, credentials source, and data are ready.
8. Run full automation suite only when user explicitly asks or release policy requires it.
9. Produce script review package and wait for human OK/NOK when promotion is requested.
10. Produce promotion checklist and require explicit approval before any remote push, deploy, or merge.

## Change Set

Use `automation_change_set.json` as canonical state. Include:
- `repo_context`: repo path, package manager, test commands, branch info, detected conventions.
- `framework`: detected framework or Playwright TypeScript fallback.
- `branch_policy`: repo policy or fallback `develop -> master`.
- `automation_scope`: create, update, manual-only, skipped, or validation-only work.
- `test_case_mappings`: TC ID to script path, spec title, assertions, selectors, and status.
- `planned_changes`: intended file actions before edits.
- `applied_changes`: actual file actions after edits.
- `validation_results`: lint, typecheck, format check, config validation, targeted run, full run.
- `review_gates`: script review and promotion review with OK/NOK state.
- `promotion_status`: readiness for push, deploy, or merge.
- `gaps`: selector, data, environment, fixture, API, repo, or framework gaps.
- `changelog`: dated change set events.

Allowed actions are `create_script`, `update_script`, `skip_manual_only`, `mark_gap`, `refactor_approved`, and `validation_only`.

## Script Rules

- Follow existing framework before fallback.
- Use Playwright TypeScript fallback only for new/no-framework automation.
- Put TC ID in spec title, annotation, comment, or metadata.
- Derive assertions from expected results.
- Never write real secrets, tokens, passwords, cookies, or PII.
- Use fixtures/config for test data and `.env.example` for environment names.
- Use Page Object Model for repeated UI flows.
- Prefer shared auth/session fixtures when repo already uses them.
- Validate API status, payload, headers, and error bodies when relevant.
- Use DB verification only when connection and query are safe and provided.
- Avoid `waitForTimeout()` unless unavoidable; document reason in the change set.
- Keep tests deterministic, isolated, and cleanup-aware.
- Refactor minimally unless refactor is required or explicitly approved.

## Locator Policy

Default priority:
1. `getByRole()` with accessible name.
2. `getByLabel()` for form inputs.
3. `getByPlaceholder()` only when label is absent and placeholder is stable.
4. `getByText()` only when unique and stable.
5. `getByTestId()` following repo test id config.
6. Stable CSS id/class only when semantic locator is unavailable.
7. XPath only as last resort with reason.

Do not use generated class hashes, deep CSS chains, unstable ids, volatile text, or blind `.nth()` selectors. Record `selector_gap` when a stable selector is missing and suggest a testability improvement.

## Review Gates

Script review:
- Human reviews scripts, patch summary, fixtures, locators, assertions, and validation.
- Status is `OK` or `NOK`.
- On `NOK`, revise change set and scripts before promotion.

Promotion review:
- Requires script review OK, validation status known, gaps documented, branch policy known, and remote action approval state captured.
- Remote push, deploy, or merge needs explicit user approval even when readiness is `ready_to_push`, `ready_to_deploy`, or `ready_to_merge`.

Promotion states: `draft`, `script_review_required`, `revision_required`, `validation_required`, `promotion_review_required`, `ready_to_push`, `ready_to_deploy`, `ready_to_merge`, `promoted`, `blocked`.

## Outputs

Canonical output:
- `automation_change_set.json`

Human outputs:
- `automation-review.md`
- `validation-report.md`
- `patch-summary.md`
- `promotion-checklist.md`

Downstream outputs:
- `automation_status_update`
- `qa_executor_validation_request`
- `qa_reporter_automation_summary`

Status update rules:
- script created and validation passed -> candidate for `Already Automate`
- script created but validation not run -> keep `To Be Automate`, mark `script_created_unvalidated`
- blocked by gap -> keep `To Be Automate`, record gap
- `Manual Only` remains unchanged unless human changes automation status

## Resources

Read only when needed:
- `schemas/automation-change-set.schema.json`: canonical change set schema.
- `schemas/automation-review.schema.json`: script and promotion review feedback schema.
- `schemas/validation-result.schema.json`: validation command result schema.
- `schemas/automation-status-update.schema.json`: downstream automation status schema.
- `templates/`: reusable review, validation, patch, promotion, and change-set templates.
- `references/playwright-typescript.md`: fallback project structure and command patterns.
- `references/locator-policy.md`: selector priority and anti-fragile rules.
- `references/branch-policy.md`: branch and remote action policy.
- `examples/`: planner contract, executor signal, change set, and review examples.
