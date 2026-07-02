# QA API Test Cases

| tc_id | scenario | summary | method | url | header | params | body | expected_result | actual_result | test_case_status | automation_status | notes |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| TC0001 | | | GET | /path | | | | | | Untested | To Be Automate | |

## Field Rules

- `method`: `GET`, `POST`, `PUT`, `PATCH`, `DELETE`, `HEAD`, or `OPTIONS`.
- `url`: endpoint path or URL from source evidence; do not invent base URL.
- `header`: required request headers, auth scheme, content type, or header refs.
- `params`: query/path params and values or fixture refs.
- `body`: request body, payload shape, or fixture ref.
- `expected_result`: expected status code, response body, schema, side effect, or error contract.
- `actual_result` remains blank until executor result or explicit user instruction exists.
- `test_case_status`: `Untested`, `On Progress`, `Passed`, `Failed`, `Blocked`, or `Retest`.
- `automation_status`: `Manual Only`, `To Be Automate`, `Automation Implemented`, or `Already Automate`.
