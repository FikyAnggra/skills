# Manual Execution Result Example

Package ID: `QA-PLAN-AUTH-LOGIN-v0.2`
Project: Customer Portal
Feature: Login
Environment: Staging web, build 2026.06.11.1
Executed by: QA Analyst
Execution date: 2026-06-11

## Planner and Executor Style Input

Planning state ref: `planning_state.json`
Execution result ref: `execution_result.json`
Coverage ref: `planning_state.coverage_map`
Risk ref: `planning_state.risk_analysis`
Approval metadata: planning approved by Product Owner on 2026-06-10, execution accepted by QA Lead on 2026-06-11.

## Manual Results

| TC ID | Scenario | Priority | Status | Expected Result | Actual Result | Evidence/Notes |
| --- | --- | --- | --- | --- | --- | --- |
| TC0001 | Valid user logs in | P0 - Critical | Passed | User lands on dashboard. | User landed on dashboard. | `evidence/login-success.png` |
| TC0002 | Invalid password rejected | P1 - High | Failed | Generic invalid credential message is shown. | User landed on dashboard. | `evidence/invalid-password-dashboard.png` |
| TC0003 | Forgot password email | P2 - Medium | Blocked | Reset email is sent. | Mail service unavailable. | Blocked by staging SMTP outage; no screenshot available. |
| TC0004 | Logout clears session | P1 - High | Untested | Session is cleared and login page is shown. | Not executed. | Execution window ended. |

## Reporter Notes

- TC0002 should create an issue submission package.
- TC0003 should create a blocker issue package or report gap, depending on triage policy.
- Missing evidence for TC0003 must be recorded as a report gap.
- Critical login coverage is partial because logout was untested.
