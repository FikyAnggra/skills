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

### Severity Normalization

{{severity_normalization.reason}}

### Report Gaps

{{#report_gaps}}
- {{.}}
{{/report_gaps}}
{{/issue_package.issues}}
