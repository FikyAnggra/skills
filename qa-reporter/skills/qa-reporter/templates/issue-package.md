# Issue Submission Package

Report ID: `{{report_id}}`
Package ID: `{{package_id}}`
Issue package status: `{{issue_package.status}}`

Submission gate:
- Explicit user submit request: `required`
- Package approval: `required`
- Tool/config availability: `required`
- Required fields complete: `required`
- Redaction status `passed`: `required`

{{#issue_package.issues}}
## {{issue_id}} - {{title}}

Status: `{{status}}`
Severity: `{{severity}}`
Priority: `{{priority}}`
Suggested owner: `{{suggested_owner}}`
Submission status: `{{submission_status}}`
Redaction status: `{{redaction_status}}`
Target tracker: `{{target_tracker}}`
Submission mode: `{{submission_mode}}`

### Summary

{{summary}}

### Affected Test Cases

{{#affected_test_cases}}
- {{.}}
{{/affected_test_cases}}

### Reproduction Steps

{{#reproduction_steps}}
1. {{.}}
{{/reproduction_steps}}

### Expected Result

{{expected_result}}

### Actual Result or Blocker Detail

{{actual_result}}{{blocker_detail}}

### Evidence References

{{#evidence_refs}}
- {{.}}
{{/evidence_refs}}

### Plane Routing

Workspace slug: `{{custom_fields.plane.workspace_slug}}`
Project: `{{custom_fields.plane.project_identifier}}` / `{{custom_fields.plane.project_id}}`
Work item: `{{custom_fields.plane.readable_identifier}}` / `{{custom_fields.plane.work_item_id}}`
State: `{{custom_fields.plane.state_name}}`
Type: `{{custom_fields.plane.type_name}}`
Labels: `{{custom_fields.plane.label_names}}`
Cycle: `{{custom_fields.plane.cycle_id}}`
Module: `{{custom_fields.plane.module_id}}`

### Duplicate Check

Status: `{{duplicate_check.status}}`
Query: `{{duplicate_check.query}}`

{{#duplicate_check.matched_items}}
- `{{readable_identifier}}` {{title}} - {{match_reason}}
{{/duplicate_check.matched_items}}

### Severity Normalization

{{severity_normalization.reason}}

### Report Gaps

{{#report_gaps}}
- {{.}}
{{/report_gaps}}
{{/issue_package.issues}}

