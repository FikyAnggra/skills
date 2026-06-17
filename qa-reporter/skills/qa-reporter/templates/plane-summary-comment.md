QA reporter summary for `{{report_id}}`

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
