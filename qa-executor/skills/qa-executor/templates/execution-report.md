# QA Execution Report

Execution ID: `{execution_id}`
Package ID: `{package_id}`
Environment: `{environment.name}`
Run mode: `{run_policy.mode}`
Started: `{started_at}`
Finished: `{finished_at}`
Review status: `{review_status}`

## Summary

| Metric | Count |
| --- | ---: |
| Total | `{summary.total}` |
| Selected | `{summary.selected}` |
| Passed | `{summary.passed}` |
| Failed | `{summary.failed}` |
| Blocked | `{summary.blocked}` |
| Untested | `{summary.untested}` |
| Retest | `{summary.retest}` |

## Results

| TC ID | Scenario | Mode | Status | Actual Result | Evidence | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| `{tc_id}` | `{scenario}` | `{execution_mode}` | `{test_case_status}` | `{actual_result}` | `{evidence_refs}` | `{notes}` |

## Blockers

| TC ID | Type | Reason | Blocked By |
| --- | --- | --- | --- |
| `{tc_id}` | `{blocker.blocker_type}` | `{blocker.blocker_reason}` | `{blocker.blocked_by}` |

## Retries

| TC ID | Attempt | Reason | Outcome | Evidence |
| --- | ---: | --- | --- | --- |
| `{tc_id}` | `{attempt}` | `{reason}` | `{outcome}` | `{evidence_refs}` |

## Issue Candidates

| Candidate ID | TC ID | Severity | Title | Evidence |
| --- | --- | --- | --- | --- |
| `{issue_candidate_id}` | `{tc_id}` | `{severity_suggestion}` | `{title}` | `{evidence_refs}` |

## Evidence And Redaction

- Redaction status: `{evidence.redaction_status}`
- Evidence gaps: `{evidence.evidence_gaps}`
- Large files are referenced by path or URL, not embedded.

## Source Updates

- Source update status: `{report_outputs.source_update_status}`
- Source update summary: `{report_outputs.source_update_summary}`
- Every selected writable-source test case should have a source update record or source update gap before this report is treated as review-ready.

## Review

- Decision: `{review_status}`
- Reviewer: `{reviewer}`
- Reviewed at: `{reviewed_at}`
- Notes: `{decision_note}`
