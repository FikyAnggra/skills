# QA Automation

`qa-automation` is a portable QA automation package compatible with Codex and Claude-style project workflows. It creates or updates automation scripts from approved QA inputs through a governed `automation_change_set.json` workflow, then can sync automation status and workflow summaries through Notion and Plane when those sources are provided.

## Contents

- `.codex-plugin/plugin.json`: Codex plugin wrapper.
- `.claude-plugin/plugin.json`: Claude-facing metadata wrapper that points to the same canonical skill.
- `skills/qa-automation/SKILL.md`: canonical portable automation instructions.
- `skills/qa-automation/schemas/`: JSON schemas for change sets, reviews, validation results, automation status updates, Notion sync records, and Plane sync records.
- `skills/qa-automation/templates/`: reusable change set, review, validation, patch, promotion checklist, Notion, Plane, and bridge templates.
- `skills/qa-automation/references/`: Playwright TypeScript fallback, locator policy, branch policy, Notion MCP, Plane MCP, and Notion/Plane bridge rules.
- `skills/qa-automation/examples/`: planner contract, executor signal, change set, review, Notion, Plane, and bridge examples.
- `skills/qa-automation/agents/openai.yaml`: Codex UI metadata.

## Inputs

Use planner contracts for `create_scripts`, `update_scripts`, `manual_only`, approved test cases, and automation gaps. Use executor signals for selector gaps, flaky technical failures, API/data gaps, and update candidates. Manual requests can provide test case text, existing script paths, repo path, framework note, branch policy, or validation command.

Use Notion input when a request provides a Notion page, database, data source, row/page id, URL, or MCP payload. Use Plane input when a request provides a Plane work item/card/sub-work-item, readable id, URL, UUID, comment, page/wiki, or MCP payload. When Plane contains Notion links, `qa-automation` can bridge both systems while keeping `automation_change_set.json` canonical.

## Framework Policy

Follow the target repository framework and style first. If no repo/framework exists, use the Playwright TypeScript fallback in `references/playwright-typescript.md`.

## Locator Policy

Prefer semantic Playwright locators: role, label, placeholder, stable text, configured test id, stable CSS, then XPath as last resort. Record `selector_gap` instead of inventing selectors.

## Validation Policy

Run available light validation: format check, lint, typecheck, and config validation. Run targeted automation when environment and data are ready. Run full automation only by explicit user request or release policy.

## Review and Promotion

Script review and promotion review use OK/NOK gates. Promotion can become ready for push, deploy, or merge, but remote actions still require explicit user approval.

## Notion and Plane Sync

Notion can be used as the test case/status source. Automation statuses are `Manual Only`, `To Be Automate`, `Automation Implemented`, and `Already Automate`. Script creation/update moves a case to `Automation Implemented`; `Already Automate` requires script review `OK` and relevant validation `passed`.

Plane can be used as the workflow surface. It supports managed comments, description summaries, links, worklogs, pages/wiki, and dynamic state sync when a valid mapping is provided. There is no hard-coded Plane automation workflow; states are read from the project before movement.

Create/update operations are allowed when request and write policy are clear. Delete/archive operations in Notion or Plane always require explicit approval after target impact is known. Gap follow-up items are not created automatically.

## Outputs

Canonical output is `automation_change_set.json`. Human outputs include `automation-review.md`, `validation-report.md`, `patch-summary.md`, and `promotion-checklist.md`. Downstream outputs include `automation_status_update`, `qa_executor_validation_request`, and `qa_reporter_automation_summary`. Managed sync outputs include Notion sync records, Plane sync records, and optional Notion/Plane bridge records.
