# QA Test Report

Report ID: `{{report_id}}`
Package ID: `{{package_id}}`
Status: `{{report_status}}`
Environment: `{{metadata.environment}}`
Version: `{{metadata.report_version}}`

## Executive Summary

{{summary.objective}}

Overall status: `{{summary.overall_status}}`

Key findings:
{{#summary.key_findings}}
- {{.}}
{{/summary.key_findings}}

Report gaps:
{{#summary.report_gaps}}
- {{.}}
{{/summary.report_gaps}}

## Execution Metrics

| Metric | Value |
| --- | ---: |
| Total test cases | {{execution_metrics.total_test_cases}} |
| Selected test cases | {{execution_metrics.selected_test_cases}} |
| Executed test cases | {{execution_metrics.executed_test_cases}} |
| Passed | {{execution_metrics.passed}} |
| Failed | {{execution_metrics.failed}} |
| Blocked | {{execution_metrics.blocked}} |
| Untested | {{execution_metrics.untested}} |
| Retest | {{execution_metrics.retest}} |
| Pass rate | {{execution_metrics.pass_rate}}% |
| Execution completion rate | {{execution_metrics.execution_completion_rate}}% |
| Critical/High issue count | {{execution_metrics.critical_high_issue_count}} |
| Open blocker count | {{execution_metrics.open_blocker_count}} |

Metric gaps:
{{#execution_metrics.metric_gaps}}
- {{.}}
{{/execution_metrics.metric_gaps}}

## Failed and Blocked Details

{{#issue_package.issues}}
### {{issue_id}} - {{title}}

Severity: `{{severity}}`
Priority: `{{priority}}`
Status: `{{status}}`
Affected test cases: `{{affected_test_cases}}`

Expected result:
{{expected_result}}

Actual result or blocker detail:
{{actual_result}}{{blocker_detail}}

Evidence refs:
{{#evidence_refs}}
- {{.}}
{{/evidence_refs}}

Gaps:
{{#report_gaps}}
- {{.}}
{{/report_gaps}}
{{/issue_package.issues}}

## Coverage and Risk

Coverage status: `{{coverage.coverage_status}}`
Critical coverage status: `{{coverage.critical_coverage_status}}`
Risk level: `{{risk.risk_level}}`

Coverage source refs:
{{#coverage.source_refs}}
- {{.}}
{{/coverage.source_refs}}

Open risks:
{{#risk.open_risks}}
- {{.}}
{{/risk.open_risks}}

## Evidence References

{{#summary.evidence_refs}}
- {{.}}
{{/summary.evidence_refs}}

## Sign-Off Recommendation

Recommendation: `{{signoff_recommendation.recommendation}}`
Rule: `{{signoff_recommendation.rule}}`
Reason: {{signoff_recommendation.reason}}
