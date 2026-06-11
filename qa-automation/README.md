# QA Automation

`qa-automation` is a portable QA automation package compatible with Codex and Claude-style project workflows. It creates or updates automation scripts from approved QA inputs through a governed `automation_change_set.json` workflow.

## Contents

- `.codex-plugin/plugin.json`: Codex plugin wrapper.
- `.claude-plugin/plugin.json`: Claude-facing metadata wrapper that points to the same canonical skill.
- `skills/qa-automation/SKILL.md`: canonical portable automation instructions.
- `skills/qa-automation/schemas/`: JSON schemas for change sets, reviews, validation results, and automation status updates.
- `skills/qa-automation/templates/`: reusable change set, review, validation, patch, and promotion checklist templates.
- `skills/qa-automation/references/`: Playwright TypeScript fallback, locator policy, and branch policy.
- `skills/qa-automation/examples/`: planner contract, executor signal, change set, and review examples.
- `skills/qa-automation/agents/openai.yaml`: Codex UI metadata.

## Inputs

Use planner contracts for `create_scripts`, `update_scripts`, `manual_only`, approved test cases, and automation gaps. Use executor signals for selector gaps, flaky technical failures, API/data gaps, and update candidates. Manual requests can provide test case text, existing script paths, repo path, framework note, branch policy, or validation command.

## Framework Policy

Follow the target repository framework and style first. If no repo/framework exists, use the Playwright TypeScript fallback in `references/playwright-typescript.md`.

## Locator Policy

Prefer semantic Playwright locators: role, label, placeholder, stable text, configured test id, stable CSS, then XPath as last resort. Record `selector_gap` instead of inventing selectors.

## Validation Policy

Run available light validation: format check, lint, typecheck, and config validation. Run targeted automation when environment and data are ready. Run full automation only by explicit user request or release policy.

## Review and Promotion

Script review and promotion review use OK/NOK gates. Promotion can become ready for push, deploy, or merge, but remote actions still require explicit user approval.

## Outputs

Canonical output is `automation_change_set.json`. Human outputs include `automation-review.md`, `validation-report.md`, `patch-summary.md`, and `promotion-checklist.md`. Downstream outputs include `automation_status_update`, `qa_executor_validation_request`, and `qa_reporter_automation_summary`.
