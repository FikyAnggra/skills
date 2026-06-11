# QA Execution Report

Execution ID: `EXEC-20260611-001`
Package ID: `EXEC-PKG-AUTH-LOGIN-20260611`
Environment: Staging, build `2026.06.11.1`
Run mode: `targeted`
Started: `2026-06-11T10:00:00+07:00`
Finished: `2026-06-11T10:20:00+07:00`
Review status: `OK`

## Summary

| Metric | Count |
| --- | ---: |
| Total | 3 |
| Selected | 3 |
| Passed | 1 |
| Failed | 1 |
| Blocked | 1 |
| Untested | 0 |
| Retest | 0 |

Pass rate: 33.33%
Completion rate: 100%

## Results

| TC ID | Scenario | Mode | Status | Actual Result | Evidence | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| TC0001 | Valid active user logs in | ui | Passed | User landed on dashboard. | `evidence/TC0001-dashboard.png`, `evidence/TC0001-trace.zip` | Passed after one technical retry. |
| TC0002 | Invalid password is rejected | hybrid | Failed | User landed on dashboard. | `evidence/TC0002-invalid-password-dashboard.png`, `evidence/TC0002-trace.zip` | Business assertion failure, no retry performed. |
| TC0003 | Product creation persists to DB | hybrid | Blocked | Could not verify DB persistence because read-only DB credentials were unavailable. | `evidence/TC0003-api-create-summary.json` | Blocked by credential gap, not a product assertion failure. |

## Evidence Highlights

### UI Evidence

- `TC0001`: dashboard screenshot and trace captured.
- `TC0002`: dashboard screenshot captured after invalid password submit.

### API Evidence

- `TC0002`: `POST https://staging.example.com/api/auth/login` returned `200`; request password and response token redacted.
- `TC0003`: `POST https://staging.example.com/api/products` returned `201`; response body summarized.

### DB Evidence

- `TC0003`: DB check `Verify product created` was not run because approved read-only credential was unavailable.

### Retry Evidence

| TC ID | Attempt | Reason | Outcome | Evidence |
| --- | ---: | --- | --- | --- |
| TC0001 | 1 | Temporary timeout waiting for dashboard route. | Second attempt passed; recorded as technical flake. | `evidence/TC0001-retry-1-timeout.log` |

## Blockers

| TC ID | Type | Reason | Blocked By |
| --- | --- | --- | --- |
| TC0003 | credential | Approved read-only DB credential was not available in the execution environment. | None |

## Issue Candidates

| Candidate ID | TC ID | Severity | Title | Human Review |
| --- | --- | --- | --- | --- |
| ISSUE-CAND-TC0002 | TC0002 | High | Invalid password allows dashboard access | Required |
| ISSUE-CAND-TC0003 | TC0003 | Medium | Product persistence test blocked by missing DB credential | Required |

## Downstream Handoff

- `qa_reporter_handoff`: reference `execution_result.json`, summary, issue candidates, evidence refs, and OK review metadata.
- `qa_automation_signal`: includes one flaky technical failure, one data gap, and one update candidate. This is not a script generation request by itself.

## Redaction

Redaction status: `passed`. Passwords, authorization values, tokens, and credential details are represented as `[REDACTED]` or source refs only.
