<!-- qa-executor:notion-execution:EXEC-20260617-002:notion-execution-page-id -->

# QA Execution Result

Execution ID: `EXEC-20260617-002`
Package ID: `EXEC-PKG-NOTION-AUTH-20260617`
Source Notion ref: `Auth Test Cases`
Environment: `Staging`
Run mode: `targeted`
Started: `2026-06-17T11:00:00+07:00`
Finished: `2026-06-17T11:10:00+07:00`
Review status: `pending`

## Summary

| Total | Selected | Passed | Failed | Blocked | Untested | Retest |
| ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| 2 | 2 | 1 | 1 | 0 | 0 | 0 |

## Results

| TC ID | Scenario | Mode | Status | Actual | Evidence |
| --- | --- | --- | --- | --- | --- |
| TC0001 | Valid active user logs in | ui | Passed | User landed on dashboard. | `evidence/notion/TC0001-dashboard.png` |
| TC0002 | Invalid password is rejected | hybrid | Failed | User landed on dashboard. | `evidence/notion/TC0002-invalid-password-dashboard.png` |

## Issue Candidates

| Candidate ID | TC ID | Severity | Title | Human Review |
| --- | --- | --- | --- | --- |
| ISSUE-CAND-TC0002 | TC0002 | High | Invalid password allows dashboard access | required |

## Handoffs

- Reporter handoff: `execution_result.qa_reporter_handoff`
- Automation signal: `execution_result.qa_automation_signal`

## Notion Sync

- Source database update: `true`
- Execution page sync: `true`
- Execution database sync: `false`
- Comment sync: `false`
- Issue candidate sync: `false`
- Sync status: `synced`

## Redaction

Redaction status: `passed`. Credentials, authorization values, tokens, cookies, session ids, and PII are redacted before Notion sync.
