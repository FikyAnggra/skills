<!-- qa-executor:execution:<execution_id>:<plane_work_item_id> -->

# QA Execution Sync

Execution ID: `<execution_id>`
Package ID: `<package_id>`
Plane item: `<readable_identifier>`
Run mode: `<run_policy.mode>`
Environment: `<environment.name>`
Started: `<started_at>`
Finished: `<finished_at>`
Review status: `<review_status>`

## Status Mapping

| Event | Plane State |
| --- | --- |
| Start | `<plane_status_mapping.on_start>` |
| All passed | `<plane_status_mapping.on_all_passed>` |
| Any failed | `<plane_status_mapping.on_any_failed>` |
| Blocked | `<plane_status_mapping.on_blocked>` |
| Partial | `<plane_status_mapping.on_partial>` |
| Cancelled | `<plane_status_mapping.on_cancelled>` |

## Summary

| Total | Selected | Passed | Failed | Blocked | Untested | Retest |
| ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| `<summary.total>` | `<summary.selected>` | `<summary.passed>` | `<summary.failed>` | `<summary.blocked>` | `<summary.untested>` | `<summary.retest>` |

## Results

| TC ID | Scenario | Mode | Status | Actual Result | Evidence |
| --- | --- | --- | --- | --- | --- |
| `<tc_id>` | `<scenario>` | `<execution_mode>` | `<test_case_status>` | `<actual_result>` | `<evidence_refs>` |

## Blockers And Retries

- Blockers: `<blocker_summary>`
- Retries: `<retry_summary>`

## Issue Candidates

| Candidate ID | TC ID | Severity | Title | Human Review |
| --- | --- | --- | --- | --- |
| `<issue_candidate_id>` | `<tc_id>` | `<severity_suggestion>` | `<title>` | `required` |

## Handoffs

- Reporter handoff: `<qa_reporter_handoff_ref>`
- Automation signal: `<qa_automation_signal_ref>`

## Plane Sync

- Status before: `<plane_sync.status_before>`
- Status after: `<plane_sync.status_after>`
- Transition reason: `<plane_sync.status_transition_reason>`
- Worklog: `<plane_sync.worklog_id>`
- Sync status: `<plane_sync.sync_status>`
- Sync errors: `<plane_sync.sync_errors>`

## Redaction

Redaction status: `<redaction_status>`

Evidence is stored as refs or links. Secrets, tokens, cookies, credentials, authorization values, session ids, and PII must be redacted before Plane sync.
