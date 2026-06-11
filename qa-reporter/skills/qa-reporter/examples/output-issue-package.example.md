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

### Severity Normalization

High severity because a major authentication flow accepts invalid credentials with limited workaround.

### Submission Status

Not submitted. Issue submission requires explicit user request after package approval and tool/config confirmation.
