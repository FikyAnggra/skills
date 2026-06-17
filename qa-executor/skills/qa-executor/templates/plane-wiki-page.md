# QA Execution Evidence

Plane item: `<readable_identifier>`
Execution ID: `<execution_id>`
Package ID: `<package_id>`
Environment: `<environment.name>`
Run mode: `<run_policy.mode>`
Review status: `<review_status>`

This Plane wiki/page output is opt-in only. Create or update it only when `plane_write_policy.wiki_sync = true` or the user explicitly asks to publish/update wiki output.

## Summary

| Total | Selected | Passed | Failed | Blocked | Untested | Retest |
| ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| `<summary.total>` | `<summary.selected>` | `<summary.passed>` | `<summary.failed>` | `<summary.blocked>` | `<summary.untested>` | `<summary.retest>` |

## Test Results

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

## Downstream Refs

- Execution result: `<execution_result_ref>`
- Reporter handoff: `<qa_reporter_handoff_ref>`
- Automation signal: `<qa_automation_signal_ref>`

## Redaction

Redaction status: `<redaction_status>`

Do not paste raw logs, raw payloads, secrets, credentials, cookies, tokens, authorization values, session ids, or PII into this page.
