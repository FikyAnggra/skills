# QA Test Cases

| tc_id | scenario | summary | test_type | priority | pre_conditions | test_steps | test_data | expected_result | actual_result | test_case_status | automation_status | notes |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| TC0001 | | | Functional | P2 - Medium | | 1. | | | | Untested | Manual Only | |

## Test Variables

Use variables in `pre_conditions`, `test_steps`, `test_data`, and `expected_result` with `{{variable_name}}` placeholders.

| tc_id | variable | description | source | required | example_or_placeholder | sensitive | preparation_notes |
|---|---|---|---|---|---|---|---|
| TC0001 | `{{valid_user_email}}` | Valid user email for login or account flow | needs_user_or_fixture | true | `{{valid_user_email}}` | false | Prepare before execution |

## Field Rules

- `priority`: `P0 - Critical`, `P1 - High`, `P2 - Medium`, or `P3 - Low`.
- `test_case_status`: `Untested`, `On Progress`, `Passed`, `Failed`, `Blocked`, or `Retest`.
- `automation_status`: `Manual Only`, `To Be Automate`, `Automation Implemented`, or `Already Automate`.
- `actual_result` remains blank until executor result or explicit user instruction exists.
- `test_steps` should be ordered and observable.
- `test_variables` must list all required accounts, payloads, ids, files, dates, tokens, fixtures, and expected computed values. Use placeholders instead of invented values.
