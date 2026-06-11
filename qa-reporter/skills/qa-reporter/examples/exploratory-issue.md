# Exploratory Issue Example

Project: Customer Portal
Feature: Login
Environment: Staging web, build 2026.06.11.1
Found by: QA Analyst
Found at: 2026-06-11T14:30:00+07:00

## Finding

Title: Login accepts invalid password after prior successful session

## Steps to Reproduce

1. Log in with a valid user.
2. Log out from the dashboard.
3. Return to the login page.
4. Enter the same username with an invalid password.
5. Submit the login form.

## Expected Result

A generic invalid credential message is shown and the user remains on the login page.

## Actual Result

The user lands on the dashboard.

## Evidence

- `evidence/exploratory-invalid-password.png`
- Browser console contained no relevant error.

## Impact Hint

Major authentication flow defect. Suggested severity: High.

## Reporter Treatment

Create a bug report and issue submission package. Redaction status can pass only after confirming the screenshot does not expose personal data or tokens.
