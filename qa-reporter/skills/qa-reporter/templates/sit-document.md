# SIT Document

Report ID: `{{report_id}}`
Package ID: `{{package_id}}`
Status: `{{sit_document.status}}`
Environment: `{{sit_document.environment}}`

## Objective

{{sit_document.objective}}

## Scope

{{#sit_document.scope}}
- {{.}}
{{/sit_document.scope}}

## Execution Summary

{{sit_document.execution_summary}}

## Coverage Summary

{{sit_document.coverage_summary}}

## Defect Summary

{{sit_document.defect_summary}}

## Known Limitations

{{#sit_document.known_limitations}}
- {{.}}
{{/sit_document.known_limitations}}

## Open Risks

{{#sit_document.open_risks}}
- {{.}}
{{/sit_document.open_risks}}

## Evidence References

{{#sit_document.evidence_refs}}
- {{.}}
{{/sit_document.evidence_refs}}

## Conclusion

{{sit_document.conclusion}}

## Sign-Off Recommendation

Recommendation: `{{signoff_recommendation.recommendation}}`
Rule: `{{signoff_recommendation.rule}}`
Reason: {{signoff_recommendation.reason}}
