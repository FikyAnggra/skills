# QA Reporter Handoff Contract

Report ID: `{{report_id}}`
Package ID: `{{package_id}}`
Source: `{{custom_fields.plane.source_work_item.readable_identifier}}`
Sub-work-item: `{{custom_fields.plane.source_sub_work_item.readable_identifier}}`

## Scope

{{summary.objective}}

## Work Item Context

- Parent work item: `{{custom_fields.plane.work_item_scope.parent_readable_identifier}}`
- Target scope: `{{custom_fields.plane.work_item_scope.scope_type}}`
- Batch status: `{{custom_fields.plane.work_item_scope.batch_processing_status}}`

## Requested Outputs

{{#custom_fields.requested_outputs}}
- {{.}}
{{/custom_fields.requested_outputs}}

## Execution Summary

{{summary.overall_status}}

## Metrics

- Passed: {{execution_metrics.passed}}
- Failed: {{execution_metrics.failed}}
- Blocked: {{execution_metrics.blocked}}
- Untested: {{execution_metrics.untested}}
- Retest: {{execution_metrics.retest}}

## Issue Handoff

{{#issue_package.issues}}
- {{issue_id}}: {{title}} (`{{severity}}`, `{{priority}}`, {{submission_status}})
{{/issue_package.issues}}

## Report Gaps and Blockers

{{#summary.report_gaps}}
- {{.}}
{{/summary.report_gaps}}

## Published Artifacts

{{#publication_history}}
- {{artifact_type}}: {{url}} (`{{publish_status}}`)
{{/publication_history}}

## Next Action

{{signoff_recommendation.recommendation}} - {{signoff_recommendation.reason}}
