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


### Plane State Mapping Recommendation

Mapping approval: `{{custom_fields.plane.state_mapping.approval_status}}`
Recommendation basis: `{{custom_fields.plane.state_mapping.recommendation_basis}}`
Project: `{{custom_fields.plane.state_mapping.project_identifier}}` / `{{custom_fields.plane.state_mapping.project_id}}`

Available states:
{{#custom_fields.plane.state_mapping.available_states}}
- `{{name}}` (`{{id}}`) group=`{{group}}` sequence=`{{sequence}}` default=`{{is_default}}`
{{/custom_fields.plane.state_mapping.available_states}}

Recommended mapping:
- issue_created_state: `{{custom_fields.plane.state_mapping.issue_created_state.name}}` (`{{custom_fields.plane.state_mapping.issue_created_state.id}}`) - {{custom_fields.plane.state_mapping.issue_created_state.recommendation_reason}}
- fix_in_progress_states: `{{custom_fields.plane.state_mapping.fix_in_progress_states}}`
- fix_done_states: `{{custom_fields.plane.state_mapping.fix_done_states}}`
- rejected_or_canceled_states: `{{custom_fields.plane.state_mapping.rejected_or_canceled_states}}`
- reopened_state: `{{custom_fields.plane.state_mapping.reopened_state.name}}` (`{{custom_fields.plane.state_mapping.reopened_state.id}}`) - {{custom_fields.plane.state_mapping.reopened_state.recommendation_reason}}

State mapping must be approved by the user before qa-reporter uses any Plane state id.
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


