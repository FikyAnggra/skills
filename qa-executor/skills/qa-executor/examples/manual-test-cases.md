# Manual Test Cases Example

Source: Human-created Markdown table normalized by `qa-executor`.

Manual defaults applied:
- Missing `Priority` becomes `P2 - Medium`.
- Missing `Test Case Status` becomes `Untested`.
- Run mode becomes `targeted` when user asks to run the provided cases.
- All provided cases are selected unless user narrows scope.

| TC ID | Scenario | Pre-Conditions | Test Step | Test Data | Expected Result | Priority | Test Case Status |
| --- | --- | --- | --- | --- | --- | --- | --- |
| TC0001 | Valid active user logs in | Staging web is reachable. Active user exists. | Open login page, enter active user email and password, submit. | `active_user@example.com`, password from approved secret source | User lands on dashboard. | P0 - Critical | Untested |
| TC0002 | Invalid password is rejected | Staging web is reachable. Active user exists. | Open login page, enter active user email and invalid password, submit. | `active_user@example.com`, invalid password | Generic invalid credential message is shown. | P1 - High | Untested |
| TC0003 | Product creation persists to DB | User can log in and product API is reachable. | Create product by API, verify product row exists in DB, view product in UI. | SKU `SKU-EXEC-001` | Product exists in DB and appears in product list. | P2 - Medium | Untested |

## Normalized Notes

- `TC0001` execution mode can be `ui`.
- `TC0002` execution mode can be `hybrid` when UI evidence and API response are both available.
- `TC0003` execution mode can be `hybrid` but becomes `Blocked` if DB access is unavailable.
