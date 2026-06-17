<!-- qa-executor:execution:EXEC-20260617-001:work-item-uuid -->

# QA Execution Sync

Execution ID: `EXEC-20260617-001`
Package ID: `EXEC-PKG-ENG-42-20260617`
Plane item: `ENG-42`
Run mode: `targeted`
Environment: `Staging`
Started: `2026-06-17T10:00:00+07:00`
Finished: `2026-06-17T10:12:00+07:00`
Review status: `OK`

## Status Mapping

| Event | Plane State |
| --- | --- |
| Start | `In Progress` |
| All passed | `QA Passed` |
| Any failed | `QA Failed` |
| Blocked | `Blocked` |
| Partial | `In Progress` |
| Cancelled | `Cancelled` |

## Summary

| Total | Selected | Passed | Failed | Blocked | Untested | Retest |
| ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| 2 | 2 | 1 | 1 | 0 | 0 | 0 |

## Results

| TC ID | Scenario | Mode | Status | Actual Result | Evidence |
| --- | --- | --- | --- | --- | --- |
| TC0001 | Valid active user logs in | ui | Passed | User landed on dashboard. | `evidence/ENG-42/TC0001-dashboard.png` |
| TC0002 | Invalid password is rejected | hybrid | Failed | User landed on dashboard. | `evidence/ENG-42/TC0002-invalid-password-dashboard.png` |

## Issue Candidates

| Candidate ID | TC ID | Severity | Title | Human Review |
| --- | --- | --- | --- | --- |
| ISSUE-CAND-TC0002 | TC0002 | High | Invalid password allows dashboard access | required |

## Handoffs

- Reporter handoff: `execution_result.qa_reporter_handoff`
- Automation signal: `execution_result.qa_automation_signal`

## Plane Sync

- Status before: `Ready for QA`
- Status after: `QA Failed`
- Transition reason: `At least one selected test failed.`
- Worklog: `worklog-uuid`
- Wiki sync: skipped by policy (`wiki_sync: false`)
- Follow-up item creation: skipped by policy (`create_followup_items: false`)
- Sync status: `synced`

## Redaction

Redaction status: `passed`. Credentials, authorization values, tokens, cookies, session ids, and PII are redacted before Plane sync.
