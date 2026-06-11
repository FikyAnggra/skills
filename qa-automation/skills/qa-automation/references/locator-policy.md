# Locator Policy

Use locators that are stable, semantic, and tied to user-observable behavior.

## Priority Order

1. `getByRole()` with accessible name.
2. `getByLabel()` for form fields.
3. `getByPlaceholder()` when a label is absent and placeholder is stable.
4. `getByText()` only when text is unique and stable.
5. `getByTestId()` following repo config.
6. Stable CSS id/class only when semantic locator is unavailable.
7. XPath only as last resort with a reason in `automation_change_set.json`.

## Anti-Fragile Rules

- Do not use deep CSS chains.
- Do not use generated class hashes.
- Do not use unstable ids.
- Do not use blind `.nth()` selectors.
- Do not use volatile text when role, label, or test id exists.
- Do not invent selectors.
- Record `selector_gap` when stable locator data is missing.
- Suggest adding a stable test id for complex or dynamic elements.
