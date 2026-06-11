# QA-Automation Portable Agent Skill Design

Date: 2026-06-11
Status: Approved design draft for user review
Source artifacts:
- `QA Agent & Workflow.pdf`
- `docs/superpowers/specs/2026-06-10-qa-planner-design.md`
- `docs/superpowers/specs/2026-06-11-qa-executor-design.md`
- `docs/superpowers/specs/2026-06-11-qa-reporter-design.md`
- `qa-executor/skills/qa-executor/templates/qa-automation-signal.json`

## 1. Purpose

`qa-automation` is a portable governed automation agent skill/package. It creates or updates automation scripts from approved test cases, automation contracts, execution signals, or manual requests. It uses `automation_change_set.json` as the canonical source of truth for planned changes, applied changes, validation, review gates, promotion readiness, gaps, and status updates.

The agent follows existing repository framework and style. If no repository or framework exists, the fallback default is Playwright TypeScript.

## 2. Scope

In scope:
- Load `qa-planner` automation contract.
- Load `qa-executor` automation signal.
- Accept manual automation request, test case, existing script path, repo path, framework note, or branch policy.
- Normalize automation work into `automation_change_set.json`.
- Detect repository framework, package manager, commands, and branch policy when repo exists.
- Create or update automation scripts.
- Create or update Page Object, fixture, helper, config, or test data files when required.
- Run light validation when commands are available.
- Run targeted automation if environment and data are ready.
- Run full automation suite only on explicit user request or release policy.
- Support script review and promotion review gates.
- Produce automation status update, executor validation request, and reporter automation summary.

Out of scope:
- Creating test cases from raw requirements.
- Running manual QA/API/UI execution as `qa-executor`.
- Creating final test report, SIT document, or UAT document.
- Submitting issues to dev.
- Pushing or merging remote git branches without explicit user request/approval.
- Inventing selectors, endpoints, credentials, test data, or fixtures without a source.
- Changing tests to pass when product behavior violates expected result.

## 3. Input Paths

`qa-automation` supports three input paths.

### 3.1 Planner Automation Contract

Inputs:
- `create_scripts`
- `update_scripts`
- `manual_only`
- automation status
- priority filter
- automation gaps
- approved test cases and expected results

Rules:
- `To Be Automate` -> create script.
- `Already Automate` -> update or validate existing script.
- `Manual Only` -> skip unless human explicitly changes status.

### 3.2 Executor Automation Signal

Inputs:
- failed automation-related test cases
- flaky technical failures
- selector gaps
- API gaps
- data gaps
- update candidates

Rules:
- failed automation-related signal -> inspect/update related script if repo/script exists.
- flaky technical failure -> improve wait/assertion/test isolation only when root cause is clear.
- business failure -> do not weaken assertion or change test to pass.

### 3.3 Manual Request

Inputs:
- test case text/table
- script update request
- repo path
- existing script path
- framework note
- branch policy
- validation command

Rules:
- If repo exists, follow repo framework and style.
- If no repo/framework exists, create or propose Playwright TypeScript structure.
- If essential information is missing, ask or record gap.

## 4. Automation Change Set as Source of Truth

`automation_change_set.json` is canonical.

Top-level structure:

```json
{
  "change_set_id": "AUTO-CS-20260611-001",
  "source_agent": "qa-automation",
  "package_id": "QA-PLAN-AUTH-LOGIN-v0.2",
  "input_refs": {},
  "repo_context": {},
  "framework": {},
  "branch_policy": {},
  "automation_scope": {},
  "test_case_mappings": [],
  "planned_changes": [],
  "applied_changes": [],
  "validation_results": [],
  "review_gates": {},
  "promotion_status": {},
  "gaps": [],
  "changelog": []
}
```

Important fields:
- `repo_context`: repo path, framework detected, package manager, test commands, branch info.
- `framework`: Playwright TypeScript, Cypress TypeScript, or detected framework.
- `branch_policy`: repo policy or fallback policy.
- `automation_scope`: create, update, manual-only, skipped.
- `test_case_mappings`: TC ID -> script path -> spec name -> assertions.
- `planned_changes`: files/actions planned.
- `applied_changes`: actual patch/file changes.
- `validation_results`: lint, typecheck, static, targeted run, full run.
- `review_gates`: script review and promotion review.
- `promotion_status`: readiness for push/deploy/merge.
- `gaps`: selector, data, environment, fixture, API, repo, framework gaps.

Actions:
- `create_script`
- `update_script`
- `skip_manual_only`
- `mark_gap`
- `refactor_approved`
- `validation_only`

## 5. Default Playwright TypeScript Structure

When no repository or framework exists and user asks to generate new automation, use this default structure:

```text
automation/
  package.json
  playwright.config.ts
  tsconfig.json
  .gitignore
  tests/
    e2e/
      login.spec.ts
    api/
      auth.api.spec.ts
  pages/
    LoginPage.ts
    DashboardPage.ts
  fixtures/
    test-data/
      users.json
    auth.fixture.ts
  support/
    env.ts
    test-data.ts
    assertions.ts
  reports/
    .gitkeep
  README.md
```

Principles:
- `tests/e2e/` for UI/E2E specs.
- `tests/api/` for API specs.
- `pages/` for Page Object Model.
- `fixtures/` for Playwright fixtures and test data.
- `support/` for environment, data loader, and reusable assertions.
- Do not create real secret files. Use `.env.example` when needed.
- If repo existing structure differs, follow existing structure.

## 6. Locator Policy

Default Playwright locator priority:
1. `getByRole()` with accessible name.
2. `getByLabel()` for form input.
3. `getByPlaceholder()` when label is absent and placeholder is stable.
4. `getByText()` only when text is unique and stable.
5. `getByTestId()` following repo test id config.
6. Stable CSS id/class only when semantic locator is unavailable.
7. XPath only as last resort with reason.

Test id attribute is configurable:
- `data-testid`
- `data-test-id`
- `data-cy`
- repo-specific convention

Anti-fragile rules:
- Do not use deep CSS chains, generated class hashes, unstable ids, or blind `.nth()` selectors.
- Do not use volatile text when role/label/test id exists.
- Do not invent selectors.
- Record `selector_gap` when selector is missing.
- Suggest testability improvement such as adding stable test id for complex/dynamic elements.

## 7. Script Generation Rules

General rules:
- Follow existing framework, style, naming, folder, fixture, and helper patterns.
- Use Playwright TypeScript fallback only when repo/framework is absent or user asks for new project.
- TC ID must appear in spec title, annotation, comment, or metadata.
- Expected result drives assertion.
- Do not hardcode real secrets, token, password, cookie, or PII.
- Use fixtures/config for test data.
- Use Page Object Model for repeated UI flows.
- Use shared auth/session fixtures when appropriate.
- API checks should validate status, payload, headers, and error body when relevant.
- DB verification only if connection/query is safe and available.
- Avoid `waitForTimeout()` unless unavoidable and reason is documented.
- Tests must be deterministic and isolated.
- Destructive data setup requires cleanup or isolated fixture.
- Retry must not hide bugs.
- Record gaps: `selector_gap`, `data_gap`, `env_gap`, `fixture_gap`, `api_gap`, `repo_gap`, `framework_gap`.

Refactor policy:
- Minimal patch by default.
- Refactor only if necessary or explicitly approved.
- No broad folder restructuring unless approved.
- Preserve existing tests and helpers unless change is required.

## 8. Validation Policy

Validation tiers:

### 8.1 Static/Light Validation

Required when command exists:
- format check
- lint
- typecheck
- config validation

### 8.2 Targeted Automation Run

Preferred when environment and data are ready:
- run only tests created/updated
- run related fixture/auth setup if needed
- capture trace/report if framework supports it

### 8.3 Full Automation Run

Allowed only when:
- user explicitly asks
- release gate policy requires it
- repo policy clearly requires it

Validation result structure:

```json
{
  "validation_type": "lint|typecheck|targeted_run|full_run",
  "command": "npm run test:e2e -- login.spec.ts",
  "status": "passed|failed|skipped|blocked",
  "started_at": "...",
  "finished_at": "...",
  "summary": "...",
  "evidence_refs": []
}
```

## 9. Review Gates and Promotion

Two gates are required.

### 9.1 Script Review

Before push or promotion:
- human reviews script, patch, fixture, selector, assertion, and validation result.
- review status: `OK` or `NOK`.
- if `NOK`, revise change set and scripts.

### 9.2 Promotion Review

After script review and validation:
- determine readiness to push, deploy, or merge.
- remote-changing actions still require explicit user approval.

Branch policy:
- Follow existing repo policy.
- Fallback `develop -> master` because PDF references this flow.
- Support feature branch, PR flow, develop/main, or custom policy.
- Push/merge remote only with explicit approval.

Promotion states:
- `draft`
- `script_review_required`
- `revision_required`
- `validation_required`
- `promotion_review_required`
- `ready_to_push`
- `ready_to_deploy`
- `ready_to_merge`
- `promoted`
- `blocked`

## 10. Outputs and Downstream Flow

Canonical output:
- `automation_change_set.json`

Human outputs:
- `automation-review.md`
- `validation-report.md`
- `patch-summary.md`
- `promotion-checklist.md`

Repo outputs:
- script files
- patch proposal
- fixture/helper/page object updates
- config updates only when required

Downstream outputs:
- `automation_status_update`
- `qa_executor_validation_request`
- `qa_reporter_automation_summary`

`automation_status_update` includes:
- TC ID
- previous automation status
- new automation status
- reason
- script path
- validation result

Status update rules:
- script created + validation passed -> candidate for `Already Automate`.
- script created but validation not run -> keep `To Be Automate`, status `script_created_unvalidated`.
- script blocked by gap -> keep `To Be Automate`, gap recorded.
- `Manual Only` unchanged unless human changes automation status.

`qa_executor_validation_request` asks executor to run related test case after automation update when needed.

`qa_reporter_automation_summary` summarizes automation readiness for report/SIT/UAT.

Output safety:
- No secrets in generated files.
- Use `.env.example`, not real `.env`.
- Redact logs.
- Mark unsupported env/data gaps.

## 11. Quality Gates

Before script review:
- every planned change maps to TC ID or approved manual request
- every assertion maps to expected result
- selectors follow locator policy or gap is recorded
- no real secrets are written
- generated script follows repo style or default Playwright structure
- manual-only cases are skipped unless status changed by human
- change set is updated

Before promotion review:
- script review is OK
- required light validation ran or is explicitly skipped with reason
- targeted run ran or is blocked/skipped with reason
- gaps are documented
- status update is accurate
- branch policy is known
- remote action requires explicit approval

## 12. Portable Package Structure

Target package:

```text
qa-automation/
  .codex-plugin/
    plugin.json
  .claude-plugin/
    plugin.json
  skills/
    qa-automation/
      SKILL.md
      schemas/
        automation-change-set.schema.json
        automation-review.schema.json
        validation-result.schema.json
        automation-status-update.schema.json
      templates/
        automation-change-set.json
        automation-review.md
        validation-report.md
        patch-summary.md
        promotion-checklist.md
      references/
        playwright-typescript.md
        locator-policy.md
        branch-policy.md
      examples/
        planner-automation-contract.json
        executor-automation-signal.json
        automation-change-set.example.json
        output-automation-review.example.md
      agents/
        openai.yaml
  README.md
  CHANGELOG.md
```

Core behavior must be platform-neutral. Codex and Claude plugin wrappers are runtime packaging layers, not required for agent behavior. `CHANGELOG.md` records versioned changes, compatibility updates, schema/template changes, and migration notes.

## 13. Implementation Notes

Implementation should produce:
- portable `qa-automation` skill folder
- Codex plugin wrapper matching other QA agents
- Claude plugin wrapper matching the same package behavior
- schemas for change set, review, validation result, and status update
- templates for change set, review, validation report, patch summary, and promotion checklist
- references for Playwright TypeScript, locator policy, and branch policy
- examples for planner contract, executor signal, and automation review output
- `README.md` and `CHANGELOG.md`
- validation checks for JSON, frontmatter, placeholder scan, and required files

No platform-specific repository tooling is mandatory. If a platform cannot edit files or run validation, `qa-automation` must still produce an automation change plan, patch summary, and review package.
