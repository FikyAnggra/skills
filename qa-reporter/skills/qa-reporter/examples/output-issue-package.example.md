# Issue Submission Package

Report ID: `RPT-20260611-001`
Package ID: `QA-PLAN-AUTH-LOGIN-v0.2`
Issue package status: `ready_for_review`

Submission gate:
- Explicit user submit request: `required`
- Package approval: `required`
- Tool/config availability: `required`
- Required fields complete: `required`
- Redaction status `passed`: `required`

## ISSUE-PKG-TC0002 - Login accepts invalid password after prior session

Status: `ready_for_review`
Severity: `High`
Priority: `P1`
Suggested owner: `dev`
Submission status: `not_submitted`
Redaction status: `passed`
Target tracker: `plane`
Submission mode: `sub_work_item`
Creation strategy: `sub_work_item`
Issue kind: `bug`

### Summary

Invalid password login resulted in dashboard access after a prior successful session.

### Affected Test Cases

- TC0002

### Reproduction Steps

1. Log in with a valid user.
2. Log out from the dashboard.
3. Enter the same username with an invalid password.
4. Submit the login form.

### Expected Result

Generic invalid credential message is shown and user remains on login page.

### Actual Result

User landed on dashboard.

### Evidence References

- `evidence/invalid-password-dashboard.png`

### Bug/Issue Marker

Type applied: `true` (`Bug`)
Labels applied: `bug`, `qa-reported`, `severity-high`
Title prefix applied: `false`
Final title: `Login accepts invalid password after prior session`
Reason: Plane Bug type and bug labels are available, so no title prefix is needed.

### Plane Routing

Workspace slug: `customer-portal`
Project: `AUTH` / `00000000-0000-4000-8000-000000000001`
State: pending user-approved mapping
Type: `Bug`
Labels: `qa-reported`, `bug`, `severity-high`, `source-manual`, `needs-retest`
### Source Plane Work Item

Source role: `requirement`
Parent work item: `AUTH-42` / `00000000-0000-4000-8000-000000000042`
Source URL: `https://app.plane.so/customer-portal/projects/auth/issues/AUTH-42`
Comment sync: `created`
Description sync: `updated`
Comment URL: `https://app.plane.so/customer-portal/projects/auth/issues/AUTH-42#comment-summary`

Notion links from source:
- https://notion.so/workspace/login-test-cases-example
- https://notion.so/workspace/login-execution-report-example

Duplicate check: `passed`
### Plane State Mapping Recommendation

Mapping approval: `pending`
Recommendation basis: `metadata`
Project: `AUTH` / `00000000-0000-4000-8000-000000000001`

Available states:
- `Backlog` (`00000000-0000-4000-8000-000000000101`) group=`backlog` sequence=`100` default=`true`
- `Started` (`00000000-0000-4000-8000-000000000102`) group=`started` sequence=`300` default=`false`
- `Ready for QA` (`00000000-0000-4000-8000-000000000103`) group=`completed` sequence=`400` default=`false`
- `Canceled` (`00000000-0000-4000-8000-000000000104`) group=`cancelled` sequence=`500` default=`false`

Recommended mapping:
- issue_created_state: `Backlog` - recommended from backlog group/default marker
- fix_in_progress_states: `Started` - recommended from started group
- fix_done_states: `Ready for QA` - recommended from completed group and QA-specific name
- rejected_or_canceled_states: `Canceled` - recommended from cancelled group
- reopened_state: `Started` - recommended from started group

State mapping must be approved by the user before qa-reporter creates or syncs Plane work items.

### Severity Normalization

High severity because a major authentication flow accepts invalid credentials with limited workaround.

### Submission Status

Not submitted. Plane sub work item creation requires explicit user request after package approval, duplicate check, redaction pass, Plane project resolution, source parent resolution, approved state mapping, bug marker selection, and read-back verification.



