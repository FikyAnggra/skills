# {{artifact_title}}

Report ID: `{{report_id}}`
Package ID: `{{package_id}}`
Status: `{{report_status}}`
Recommendation: `{{signoff_recommendation.recommendation}}`

## Plane Work Item Scope

Parent work item: `{{custom_fields.plane.work_item_scope.parent_readable_identifier}}` / `{{custom_fields.plane.work_item_scope.parent_work_item_id}}`
Scope type: `{{custom_fields.plane.work_item_scope.scope_type}}`

Targets:
{{#custom_fields.plane.work_item_scope.targets}}
- `{{readable_identifier}}` / `{{work_item_id}}` parent `{{parent_readable_identifier}}` / `{{parent_work_item_id}}` state `{{state}}` status `{{processing_status}}`
{{/custom_fields.plane.work_item_scope.targets}}

## Summary

{{summary.objective}}

Overall status: {{summary.overall_status}}

Key findings:
{{#summary.key_findings}}
- {{.}}
{{/summary.key_findings}}

Report gaps:
{{#summary.report_gaps}}
- {{.}}
{{/summary.report_gaps}}

## Metrics

| Metric | Value |
| --- | ---: |
| Selected | {{execution_metrics.selected_test_cases}} |
| Executed | {{execution_metrics.executed_test_cases}} |
| Passed | {{execution_metrics.passed}} |
| Failed | {{execution_metrics.failed}} |
| Blocked | {{execution_metrics.blocked}} |
| Untested | {{execution_metrics.untested}} |
| Retest | {{execution_metrics.retest}} |
| Pass rate | {{execution_metrics.pass_rate}}% |
| Completion | {{execution_metrics.execution_completion_rate}}% |

## Source References

{{#source_refs}}
- {{source}} / {{type}} / {{role}}: {{title}} {{url}}
{{/source_refs}}

## Published Artifacts

{{#publication_history}}
- {{artifact_type}}: {{url}} (`{{publish_status}}`)
{{/publication_history}}

## Sign-Off

Rule: `{{signoff_recommendation.rule}}`
Reason: {{signoff_recommendation.reason}}
