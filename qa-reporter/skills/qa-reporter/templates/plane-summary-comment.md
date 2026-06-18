QA reporter summary for `{{report_id}}`

Source work item: `{{custom_fields.plane.source_work_item.readable_identifier}}` {{custom_fields.plane.source_work_item.url}}

Recommendation: {{signoff_recommendation.recommendation}}
Reason: {{signoff_recommendation.reason}}

Execution: {{execution_metrics.passed}} passed, {{execution_metrics.failed}} failed, {{execution_metrics.blocked}} blocked, {{execution_metrics.untested}} untested.
Critical/High issues: {{execution_metrics.critical_high_issue_count}}
Open blockers: {{execution_metrics.open_blocker_count}}

Notion report links:
{{#publication_history}}
- {{artifact_type}}: {{url}}
{{/publication_history}}

Report gaps:
{{#summary.report_gaps}}
- {{.}}
{{/summary.report_gaps}}

Notion source links:
{{#custom_fields.plane.source_work_item.notion_links}}
- {{.}}
{{/custom_fields.plane.source_work_item.notion_links}}
