# QA-Executor Portable Agent Skill Design

Date: 2026-06-11
Status: Approved design draft for user review
Source artifacts:
- `QA Agent & Workflow.pdf`
- `docs/superpowers/specs/2026-06-10-qa-planner-design.md`
- `qa-planner/skills/qa-planner/schemas/handoff-contract.schema.json`

## 1. Purpose

`qa-executor` is a portable hybrid execution agent skill/package. It executes approved QA test cases when safe and possible, generates manual execution instructions when automation is not possible, captures enterprise-grade evidence, and produces execution result packages for human review and downstream QA agents.

The agent supports API, UI, and DB verification execution. It also supports manual test case input that did not originate from `qa-planner`.

## 2. Scope

In scope:
- Load a `qa_executor` handoff contract from `qa-planner`.
- Load manually created test cases from Excel, Google Sheets, Markdown, Notion, Word/PDF tables, or plain text.
- Normalize planner-driven or manual input into an execution package.
- Resolve execution scope using run policy.
- Execute API, UI, DB, or hybrid checks when environment/tooling/data are available and safe.
- Produce manual execution instructions when automatic execution is not possible.
- Capture execution status, actual result, assertions, evidence, retries, blockers, and issue candidates.
- Produce `execution_result.json` as source of truth.
- Render Markdown report and spreadsheet-compatible status/result updates.
- Produce `qa_reporter_handoff` and `qa_automation_signal`.
- Support human `OK/NOK` review loop for execution results.

Out of scope:
- Creating test plans or test cases from raw requirements.
- Submitting final issues to dev.
- Producing final test reports, SIT documents, or UAT documents.
- Creating or updating final automation scripts.
- Changing planning approval status.
- Inventing credentials, selectors, endpoints, DB queries, fixtures, or test data without a source.

## 3. Input Paths

`qa-executor` supports two input paths.

### 3.1 Planner-Driven Input

Input from `qa-planner`:
- `qa_executor` handoff contract.
- `planning_state.test_cases`.
- Test plan environment and execution constraints.
- Test data references.
- Preconditions.
- Run policy.

### 3.2 Manual Test Case Input

Input from human-created test cases:
- Excel or Google Sheets test case matrix.
- Markdown or Notion table.
- Word/PDF table.
- Plain text test cases.

Minimum required manual fields:
- `TC ID`
- `Scenario`
- `Test Step`
- `Expected Result`

Recommended fields:
- `Pre-Conditions`
- `Test Data`
- `Priority`
- `Test Case Status`

Manual defaults:
- If user asks to run the test case, treat it as `approved_by_user` for execution.
- Missing priority defaults to `P2 - Medium`.
- Missing status defaults to `Untested`.
- Default run mode is `targeted`.
- Selected test cases are all test cases provided by the user unless user narrows scope.
- Missing expected result cannot produce deterministic pass/fail. Ask for clarification or provide manual/exploratory instructions.

## 4. Execution Result as Source of Truth

`execution_result.json` is the source of truth for execution. Test case spreadsheets, Markdown reports, Notion pages, Word files, or PDFs are rendered views.

Top-level structure:

```json
{
  "execution_id": "EXEC-20260611-001",
  "source_agent": "qa-executor",
  "package_id": "QA-PLAN-AUTH-LOGIN-v0.2",
  "planning_state_ref": "planning_state.json",
  "run_policy": {},
  "environment": {},
  "started_at": "2026-06-11T10:00:00+07:00",
  "finished_at": "2026-06-11T10:15:00+07:00",
  "summary": {
    "total": 10,
    "passed": 7,
    "failed": 1,
    "blocked": 2,
    "untested": 0,
    "retest": 0
  },
  "results": []
}
```

Per-test result structure:

```json
{
  "tc_id": "TC0001",
  "scenario": "Login with valid active user",
  "selected": true,
  "execution_mode": "api|ui|db|hybrid|manual_instruction",
  "test_case_status": "Passed",
  "actual_result": "User landed on dashboard",
  "started_at": "...",
  "finished_at": "...",
  "duration_ms": 12500,
  "steps": [],
  "assertions": [],
  "blocker": null,
  "retries": [],
  "evidence": {},
  "issue_candidate": null,
  "automation_update_signal": null
}
```

Rendered test case artifact update:
- `Actual Result` comes from `actual_result`.
- `Test Case Status` comes from `test_case_status`.
- `Notes` includes concise evidence, blocker, retry, or issue candidate summary.
- `Automation Status` is not changed unless explicit automation decision exists.

## 5. Status Model

`qa-executor` uses the existing test case status values:
- `Untested`
- `On Progress`
- `Passed`
- `Failed`
- `Blocked`
- `Retest`

Status rules:
- `Passed`: executed and all expected results/assertions matched.
- `Failed`: executed but actual result did not match expected result/assertions.
- `Blocked`: cannot execute because of failed test dependency, environment, data, access, credential, endpoint, DB, fixture, or setup problem.
- `Untested`: not selected by run policy or not attempted.
- `On Progress`: temporary status during execution.
- `Retest`: selected for rerun after previous failure/fix.

Blocked detail must be captured in `blocker`:

```json
{
  "blocker_type": "test_dependency|environment|data|access|credential|endpoint|db|fixture|setup",
  "blocker_reason": "Create product requires successful login",
  "blocked_by": ["TC0001"]
}
```

## 6. Execution Policy

`qa-executor` does not run every approved test case automatically. It resolves selection through `run_policy`.

Supported run modes:
- `initial_run`: approved test cases with `Untested` status.
- `full_regression`: all approved test cases.
- `failed_retest`: `Failed` and `Retest` test cases.
- `smoke`: priority or smoke-tagged subset.
- `targeted`: explicit test case ids.
- `automation_validation`: test cases related to automation candidates or updates.

Supported execution modes:
- `api`: API request/assertion.
- `ui`: browser or manual UI flow.
- `db`: database verification only.
- `hybrid`: API/UI/DB combination.
- `manual_instruction`: structured manual execution instructions.

Parallel policy:
- Default sequential.
- Parallel allowed only when test cases are independent, use isolated data, do not share mutable state, and run policy allows parallel.

Retry policy:
- Retry only for technical flake.
- Technical flake examples: temporary timeout, transient network failure, UI wait/race condition, temporary service unavailable.
- Do not retry business assertion failures.
- Every retry must be recorded in `retries[]` with reason, attempt number, timestamp, and outcome.

Dependency policy:
- If test case A fails and test case B depends on A, B becomes `Blocked`.
- `blocked_by` must list dependency test case ids.

## 7. Evidence Model

`qa-executor` captures enterprise-grade evidence.

Required evidence when available:
- status and actual result
- start/finish timestamp
- duration
- environment
- executor identity/tool
- step-by-step result
- assertion result
- API request summary
- API response summary
- UI screenshot, video, or trace
- DB verification result
- logs/network trace path
- retry evidence
- blocker detail
- reproduction steps for `Failed` or `Blocked`

Sensitive values must be redacted:
- token
- password
- cookie
- credential
- secret
- PII

Example API evidence:

```json
{
  "method": "POST",
  "url": "https://staging.example.com/api/auth/login",
  "request_headers": {"authorization": "[REDACTED]"},
  "request_body_summary": {"email": "active_user@example.com", "password": "[REDACTED]"},
  "response_status": 200,
  "response_body_summary": {"token": "[REDACTED]", "user_id": "usr_123"}
}
```

Example DB evidence:

```json
{
  "query_name": "Verify product created",
  "query_summary": "Count product by SKU",
  "expected": "count = 1",
  "actual": "count = 0",
  "status": "Failed"
}
```

## 8. Issue Candidate

`qa-executor` creates issue candidates for `Failed` and `Blocked`, but does not submit final issues.

Issue candidate structure:

```json
{
  "issue_candidate_id": "ISSUE-CAND-TC0002",
  "tc_id": "TC0002",
  "severity_suggestion": "High",
  "title": "Login accepts invalid password",
  "summary": "Invalid password returned dashboard instead of generic error.",
  "reproduction_steps": [],
  "expected_result": "Generic invalid credential message is shown.",
  "actual_result": "User landed on dashboard.",
  "evidence_refs": [],
  "suggested_owner": "dev",
  "requires_human_review": true
}
```

Downstream rules:
- `qa-reporter` turns issue candidates into final issue/report output after review.
- `qa-automation` may use failure/blocker signals to decide whether automation needs update.

## 9. Human Review Loop

After execution, `qa-executor` sends result package for human review.

Review status:
- `OK`: execution result accepted.
- `NOK`: execution result needs correction.

`NOK` examples:
- evidence is incomplete
- status is wrong
- actual result is unclear
- expected result was misinterpreted
- blocker reason is wrong
- retry was not documented
- redaction is incomplete

If `NOK`:
- record feedback in `execution_review_history`
- patch report/evidence if documentation is wrong
- re-run test case only when feedback requires it and environment is safe
- do not change `Passed`, `Failed`, or `Blocked` without new evidence or explicit reviewer decision

If `OK`:
- mark execution package as approved
- send `qa_reporter_handoff`
- send `qa_automation_signal` when relevant

## 10. Downstream Outputs

`qa_reporter_handoff` includes:
- execution summary
- pass/fail/block/untested/retest counts
- issue candidates
- evidence references
- risk/coverage references when available
- human approval metadata

`qa_automation_signal` includes:
- failed automation-related test cases
- flaky technical failures
- selector/API/data gaps
- candidates requiring automation script update
- note that signal is not a script generation request by itself

## 11. Quality Gates

Before review-ready output:
- every selected test case has final status
- every failed test has expected vs actual result
- every blocked test has blocker type and reason
- dependency blocks include `blocked_by`
- every retry is logged
- evidence is redacted
- issue candidates exist for `Failed` and `Blocked`
- report/spreadsheet update matches `execution_result.json`
- manual input with missing expected result is not marked pass/fail without clarification

## 12. Portable Package Structure

Target package:

```text
qa-executor/
  .codex-plugin/
    plugin.json
  .claude-plugin/
    plugin.json
  skills/
    qa-executor/
      SKILL.md
      schemas/
        execution-package.schema.json
        execution-result.schema.json
        execution-review.schema.json
        issue-candidate.schema.json
      templates/
        execution-report.md
        execution-result.json
        issue-candidate.json
        qa-reporter-handoff.json
        qa-automation-signal.json
      examples/
        manual-test-cases.md
        execution-package.example.json
        execution-result.example.json
        output-execution-report.example.md
      agents/
        openai.yaml
  README.md
  CHANGELOG.md
```

Core behavior must be platform-neutral. Codex and Claude plugin wrappers are runtime packaging layers, not required for agent behavior. `CHANGELOG.md` records versioned changes, compatibility updates, schema/template changes, and migration notes.

## 13. Implementation Notes

Implementation should produce:
- portable `qa-executor` skill folder
- Codex plugin wrapper matching `qa-planner` package pattern
- Claude plugin wrapper matching the same package behavior
- schemas for execution package/result/review/issue candidate
- templates for report/result/handoff/signal
- examples for manual and planner-driven execution
- `README.md` and `CHANGELOG.md`
- validation checks for JSON, frontmatter, placeholder scan, and required files

No platform-specific execution tool is mandatory. If a platform lacks API/UI/DB tools, `qa-executor` must still generate manual execution instructions and structured result templates.
