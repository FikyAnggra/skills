# Bug Report

Issue ID: `{{issue_id}}`
Title: {{title}}
Severity: `{{severity}}`
Priority: `{{priority}}`
Environment: `{{environment}}`

## Summary

{{summary}}

## Reproduction Steps

{{#reproduction_steps}}
1. {{.}}
{{/reproduction_steps}}

## Expected Result

{{expected_result}}

## Actual Result

{{actual_result}}

## Evidence References

{{#evidence_refs}}
- {{.}}
{{/evidence_refs}}

## Impact and Severity Reason

{{severity_normalization.reason}}

## Report Gaps

{{#report_gaps}}
- {{.}}
{{/report_gaps}}

## Plane Tracker Fields

Target tracker: `{{target_tracker}}`
Submission mode: `{{submission_mode}}`
Plane project: `{{custom_fields.plane.project_identifier}}` / `{{custom_fields.plane.project_id}}`
Plane work item: `{{custom_fields.plane.readable_identifier}}` / `{{custom_fields.plane.work_item_id}}`
Plane source sub-work-item: `{{custom_fields.plane.source_sub_work_item.readable_identifier}}` / `{{custom_fields.plane.source_sub_work_item.work_item_id}}`
Duplicate check: `{{duplicate_check.status}}`
State mapping approval: `{{custom_fields.plane.state_mapping.approval_status}}`

## Submission Gate

Do not submit this issue until the issue package is approved, redaction status is `passed`, required fields are complete, tool/config exists, and the user explicitly asks to submit.


