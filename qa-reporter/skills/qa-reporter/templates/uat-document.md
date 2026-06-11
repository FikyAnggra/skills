# UAT Document

Report ID: `{{report_id}}`
Package ID: `{{package_id}}`
Status: `{{uat_document.status}}`
Environment: `{{uat_document.environment}}`

## Objective

{{uat_document.objective}}

## Scope

{{#uat_document.scope}}
- {{.}}
{{/uat_document.scope}}

## Execution Summary

{{uat_document.execution_summary}}

## Coverage Summary

{{uat_document.coverage_summary}}

## Defect Summary

{{uat_document.defect_summary}}

## Known Limitations

{{#uat_document.known_limitations}}
- {{.}}
{{/uat_document.known_limitations}}

## Open Risks

{{#uat_document.open_risks}}
- {{.}}
{{/uat_document.open_risks}}

## Evidence References

{{#uat_document.evidence_refs}}
- {{.}}
{{/uat_document.evidence_refs}}

## Conclusion

{{uat_document.conclusion}}

## Sign-Off Recommendation

Recommendation: `{{signoff_recommendation.recommendation}}`
Rule: `{{signoff_recommendation.rule}}`
Reason: {{signoff_recommendation.reason}}
