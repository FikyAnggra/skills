<!-- qa-executor:notion-execution:<execution_id>:<notion_target_id> -->

# QA Execution Result

Execution ID: `<execution_id>`
Package ID: `<package_id>`
Source Notion ref: `<notion_source_ref>`
Environment: `<environment.name>`
Run mode: `<run_policy.mode>`
Started: `<started_at>`
Finished: `<finished_at>`
Review status: `<review_status>`

## Summary

| Total | Selected | Passed | Failed | Blocked | Untested | Retest |
| ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| `<summary.total>` | `<summary.selected>` | `<summary.passed>` | `<summary.failed>` | `<summary.blocked>` | `<summary.untested>` | `<summary.retest>` |

## Results

| TC ID | Scenario | Mode | Status | Expected | Actual | Evidence |
| --- | --- | --- | --- | --- | --- | --- |
| `<tc_id>` | `<scenario>` | `<execution_mode>` | `<test_case_status>` | `<expected_result>` | `<actual_result>` | `<evidence_refs>` |

## Blockers

| TC ID | Type | Reason | Blocked By |
| --- | --- | --- | --- |
| `<tc_id>` | `<blocker_type>` | `<blocker_reason>` | `<blocked_by>` |

## Retries

| TC ID | Attempt | Reason | Outcome | Evidence |
| --- | ---: | --- | --- | --- |
| `<tc_id>` | `<attempt>` | `<reason>` | `<outcome>` | `<evidence_refs>` |

## Issue Candidates

| Candidate ID | TC ID | Severity | Title | Evidence |
| --- | --- | --- | --- | --- |
| `<issue_candidate_id>` | `<tc_id>` | `<severity_suggestion>` | `<title>` | `<evidence_refs>` |

## Handoffs

- Reporter handoff: `<qa_reporter_handoff_ref>`
- Automation signal: `<qa_automation_signal_ref>`

## Notion Sync

- Source database update: `<notion_write_policy.source_database_update>`
- Source update status: `<report_outputs.source_update_status>`
- Source update summary: `<report_outputs.source_update_summary>`
- Execution database sync: `<notion_write_policy.execution_database_sync>`
- Comment sync: `<notion_write_policy.comment_sync>`
- Issue candidate sync: `<notion_write_policy.issue_candidate_sync>`
- Sync status: `<notion_sync.sync_status>`
- Sync errors: `<notion_sync.sync_errors>`

## Redaction

Redaction status: `<redaction_status>`

Evidence is stored as refs or links. Do not paste raw logs, raw payloads, secrets, credentials, cookies, tokens, authorization values, session ids, or PII into this page.
