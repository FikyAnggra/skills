# Coverage and Risk Summary

Report ID: `{{report_id}}`
Package ID: `{{package_id}}`

## Coverage

Coverage status: `{{coverage.coverage_status}}`
Critical coverage status: `{{coverage.critical_coverage_status}}`
Covered requirements: {{coverage.covered_requirements}} / {{coverage.total_requirements}}

Coverage gaps:
{{#coverage.coverage_gaps}}
- {{.}}
{{/coverage.coverage_gaps}}

Coverage source refs:
{{#coverage.source_refs}}
- {{.}}
{{/coverage.source_refs}}

## Risk

Risk level: `{{risk.risk_level}}`

Open risks:
{{#risk.open_risks}}
- {{.}}
{{/risk.open_risks}}

Mitigations:
{{#risk.mitigations}}
- {{.}}
{{/risk.mitigations}}

Risk source refs:
{{#risk.source_refs}}
- {{.}}
{{/risk.source_refs}}

## Sign-Off Impact

Recommendation: `{{signoff_recommendation.recommendation}}`
Reason: {{signoff_recommendation.reason}}
