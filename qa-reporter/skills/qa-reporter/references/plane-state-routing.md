# Plane State Routing for QA Reporter

Use this reference whenever input contains Plane context: Plane, Plane.so, work item Plane, card Plane, readable issue id such as `DKI-179`, Plane URL/UUID, or a request to read, analyze, create, or update a report from a Plane work item.

## Intake

Always read the Plane work item before deciding what to generate.

Read:
- title
- description
- comments
- links
- attachments
- child/sub work items
- relations
- Notion links
- evidence references
- current state
- QA Executor testing summary if present

Normalize the readable issue id, URL, UUID, project id, state id/name, evidence refs, and Notion refs into `report_state.source_refs` and `custom_fields.plane.source_work_item`.

## Automatic Start Gate

QA Reporter may automatically start report generation only when current Plane state is:
- `Need Issue Report`
- `Ready to Report`

For every other state, ask:

```text
Work item saat ini berada di state `{current_state}`. Workflow QA Reporter normal hanya berjalan otomatis dari `Need Issue Report` atau `Ready to Report`. Apakah saya boleh lanjut membuat report dan memindahkan state sesuai workflow?
```

Do not generate reports, move state, or create bug work items before confirmation when the state is outside the supported routing states.

## State: Need Issue Report

Condition:
- QA Executor finished testing.
- At least one bug, issue, blocker, defect, or problem exists.
- QA Executor moved the work item to `Need Issue Report`.

Actions:
1. Fetch the work item and source context.
2. Resolve the `Generating Issue Report` target state.
3. Move the work item to `Generating Issue Report`.
4. Read back the Plane work item and verify current state is `Generating Issue Report`.
5. If the state move fails or cannot be verified, stop. Do not create report artifacts while the work item is still in `Need Issue Report`.
6. Create issue report from work item description, comments, links, attachments/evidence, QA Executor result, failed/blocked test cases, and discovered issues.
7. Normalize each issue:
   - title
   - severity
   - priority
   - affected test case
   - expected result
   - actual result
   - reproduction steps
   - evidence refs
   - environment
   - impact
   - report gaps
8. Render:
   - `report_state`
   - issue report
   - issue submission package
   - issue candidates
   - severity/priority recommendation
   - evidence summary
   - report gaps
9. Resolve the `Need Review Issue Report` target state.
10. Move work item to `Need Review Issue Report`.
11. Read back the Plane work item and verify current state is `Need Review Issue Report`.
12. If the state move fails or cannot be verified, keep the report artifacts in draft/review-pending state, record the transition failure in `plane_state_routing.state_transition_history`, and report the blocker.
13. Add Plane comment summary that the issue report is ready for review only after the state is verified as `Need Review Issue Report`.

## State: Generating Issue Report

Condition:
- QA Reporter is generating or revising issue report.

Actions:
1. Continue issue report generation.
2. Apply human NOK feedback when present.
3. Do not change source execution result unless evidence or reviewer decision supports it.
4. Move state to `Need Review Issue Report` after completion.
5. Read back the Plane work item and verify current state is `Need Review Issue Report` before presenting the issue report as complete.

## State: Need Review Issue Report

Condition:
- Issue report is generated.
- Human review is required.

NOK actions:
1. Record review feedback.
2. Move work item to `Generating Issue Report`.
3. Revise issue report according to feedback.
4. Move work item back to `Need Review Issue Report`.

OK actions:
1. Run the Approval Blocking-Info Guard before approving. If unresolved blockers such as missing env, email, OTP, test data, access, evidence, expected result, or open blocking questions exist, ask a second explicit confirmation and stop until the user confirms or supplies the missing information.
2. Mark issue report as approved only after the guard passes.
3. Treat user input such as `approve`, `approved`, or `OK` as approval plus authorization to create the approved Plane bug/issue items for this route only after the guard passes.
4. Resolve the `Backlog Issue` target state for created bug/issue items.
5. If the issue came from this Plane source work item, create a sub work item under the source work item for each approved issue candidate. Set each sub work item state to `Backlog Issue`.
6. If there is no source Plane work item, create normal bug/issue Plane work items in `Backlog Issue`.
7. Mark each new item as bug/issue:
   - use type `Bug` when available
   - use or create label `bug`
   - use `qa-reported` label when available
   - fallback title prefix `[Bug]` or `[Issue]`
8. Read back every created work item or sub work item and verify title, parent/source relation when applicable, bug marker, and state `Backlog Issue`.
9. Save bug/issue work item links back to report source.
10. If the issue report source is a Plane work item, resolve `Backlog Test`, move the source work item from `Need Review Issue Report` to `Backlog Test`, and verify the state by read-back.
11. Add Plane comment summary to source work item with created bug/sub-work-item links and source state transition result.

Do not submit bug/issue work items to Plane before human review `OK`.

If `Backlog Issue` or `Backlog Test` is missing, run state discovery and ask the user to map the target state. Do not create issue items until `Backlog Issue` is resolved. Do not move the source work item until `Backlog Test` is resolved and all created issue items are verified.

## State: Ready to Report

Condition:
- QA Executor finished testing.
- No bug, issue, blocker, defect, or problem exists.
- QA Executor moved the work item to `Ready to Report`.

Actions:
1. Fetch the work item and source context.
2. Move the work item to `Generating Report`.
3. Create testing report from description, comments, links, attachments/evidence, QA Executor result, test case status, coverage, risk, and environment.
4. Calculate metrics:
   - total test case
   - selected test case
   - executed test case
   - passed
   - failed
   - blocked
   - untested
   - retest
   - pass rate
   - execution completion rate
   - coverage status
5. Generate sign-off recommendation:
   - `Go`
   - `Conditional Go`
   - `No-Go`
   - `Not Assessed`
6. Render:
   - `report_state`
   - testing report
   - execution summary
   - coverage/risk summary
   - SIT/UAT summary when needed
   - sign-off recommendation
   - report gaps
7. Move work item to `Need Review Report`.
8. Add Plane comment summary that the testing report is ready for review.

## State: Generating Report

Condition:
- QA Reporter is generating or revising testing report.

Actions:
1. Continue testing report generation.
2. Apply human NOK feedback when present.
3. Do not change source execution result unless evidence or reviewer decision supports it.
4. Move state to `Need Review Report` after completion.

## State: Need Review Report

Condition:
- Testing report is generated.
- Human review is required.

NOK actions:
1. Record review feedback.
2. Move work item to `Generating Report`.
3. Revise testing report according to feedback.
4. Move work item back to `Need Review Report`.

OK actions:
1. Run the Approval Blocking-Info Guard before approving. If unresolved blockers such as missing env, email, OTP, test data, access, evidence, expected result, or open blocking questions exist, ask a second explicit confirmation and stop until the user confirms or supplies the missing information.
2. Mark testing report as approved only after the guard passes.
3. Move work item to `Release Approval`.
4. Add Plane comment summary with:
   - report approved
   - testing summary
   - sign-off recommendation
   - report/evidence links

## State Discovery

Required target states:
- `Generating Issue Report`
- `Need Review Issue Report`
- `Backlog Issue`
- `Backlog Test`
- `Generating Report`
- `Need Review Report`
- `Release Approval`

If a required target state is unavailable by exact name, fetch/list Plane states and ask the user to map the missing target. Do not guess from partial names unless the user approves the mapping.

## Transition Verification

For Plane state transitions, update order matters:
- For `Need Issue Report`, move to `Generating Issue Report` before any issue report generation.
- After generating the issue report, move to `Need Review Issue Report` before saying the report is ready for review.

Every state move must be read back from Plane. If read-back does not show the expected target state, stop the workflow, keep artifacts as draft/review-pending, record the failed transition in `plane_state_routing.state_transition_history`, and report the blocker.

## Comments

Every completed state transition should leave a compact Plane comment with:
- source work item id
- action performed
- output generated
- report/evidence links
- review status or next action
- report gaps

Do not include secrets, tokens, raw credentials, raw PII, or unredacted evidence content.
