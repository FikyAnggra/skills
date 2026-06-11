# QA Test Report

Report ID: `RPT-20260611-001`
Package ID: `QA-PLAN-AUTH-LOGIN-v0.2`
Status: `draft`
Environment: `Staging web, build 2026.06.11.1`
Version: `0.1.0`

## Executive Summary

Summarize login feature test execution and release readiness.

Overall status: `Execution found one High severity authentication failure and one blocked email scenario.`

Key findings:
- Invalid password was accepted after prior successful session.
- Forgot password scenario was blocked by staging SMTP outage.

Report gaps:
- TC0003 has no screenshot evidence because the mail service was unavailable.

## Execution Metrics

| Metric | Value |
| --- | ---: |
| Total test cases | 4 |
| Selected test cases | 4 |
| Executed test cases | 3 |
| Passed | 1 |
| Failed | 1 |
| Blocked | 1 |
| Untested | 1 |
| Retest | 0 |
| Pass rate | 33.33% |
| Execution completion rate | 75% |
| Critical/High issue count | 1 |
| Open blocker count | 1 |

## Failed and Blocked Details

### ISSUE-PKG-TC0002 - Login accepts invalid password after prior session

Severity: `High`
Priority: `P1`
Status: `ready_for_review`
Affected test cases: `TC0002`

Expected result: Generic invalid credential message is shown and user remains on login page.

Actual result: User landed on dashboard.

Evidence refs:
- `evidence/invalid-password-dashboard.png`

## Coverage and Risk

Coverage status: `partial`
Critical coverage status: `gap`
Risk level: `High`

Coverage source refs:
- `planning_state.coverage_map`

Open risks:
- Authentication bypass risk remains open until TC0002 is fixed and retested.

## Sign-Off Recommendation

Recommendation: `No-Go`
Rule: `critical_high_or_exit_gap`
Reason: A High authentication defect and critical coverage gap remain open.
