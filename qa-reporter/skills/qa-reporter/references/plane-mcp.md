# Plane MCP Reference for QA Reporter

Use this reference only when the user asks `qa-reporter` to submit or sync approved issue packages through Plane.

## Source

Plane MCP docs: https://developers.plane.so/dev-tools/mcp-server

## Transport and Auth

Plane MCP supports four transport modes:
- HTTP with OAuth for Plane Cloud users and browser OAuth flow.
- HTTP with PAT token using `x-api-key` and `x-workspace-slug` headers.
- Local stdio using environment variables: `PLANE_API_KEY`, `PLANE_WORKSPACE_SLUG`, and optional `PLANE_BASE_URL`.
- SSE legacy for existing integrations.

Do not write tokens or workspace secrets into reporter artifacts. Mention only that Plane MCP must already be configured.

## Identifier Rules

Plane uses two identifier forms:
- Readable identifier such as `ENG-42`, used in UI, URLs, and team conversation.
- UUID, used by MCP tools for `project_id`, `work_item_id`, `cycle_id`, `module_id`, state, labels, and relations.

Resolve readable identifiers to UUIDs before update/comment/link/relation operations.

## Canonical Plane Tool Concepts

Prefer current Plane MCP work item terminology when available:
- `list_projects`, `retrieve_project`
- `list_work_items`, `search_work_items`, `create_work_item`, `retrieve_work_item`, `retrieve_work_item_by_identifier`, `update_work_item`
- `create_work_item_comment`
- `create_work_item_link`
- `create_work_item_relation`
- `list_states`, `list_labels`, `list_work_item_types`
- `list_cycles`, `add_work_items_to_cycle`
- `list_modules`, `add_work_items_to_module`
- `create_intake_work_item` when triage intake mode is requested
- `create_project_page` or `create_workspace_page` when publishing SIT/UAT summaries as Plane pages is requested

Some local MCP adapters may expose legacy issue names such as `create_issue`, `list_project_issues`, `add_issue_comment`, `add_cycle_issues`, or `add_module_issues`. Use them only as equivalent aliases when those are the tools present in the current runtime.

## State Discovery and Recommendation

Before any Plane create/update/sync action, fetch/list states for the target project and show them to the user. The local Plane MCP adapter exposes this as `list_states(project_id)`; newer Plane MCP servers may expose equivalent work item state tools.

Display each fetched state with:
- id/UUID
- name
- group/category when available
- sequence/order when available
- default marker when available
- description when available

Recommend qa-reporter intent mapping from available states:
- `issue_created_state`: prefer backlog group, default state, or lowest sequence state.
- `fix_in_progress_states`: prefer started states.
- `fix_done_states`: prefer completed states; prefer completed states with QA/retest/validation wording over generic completed states.
- `rejected_or_canceled_states`: prefer cancelled/canceled states.
- `reopened_state`: prefer started state unless user chooses another state.

If state group metadata is unavailable, make weak recommendations from state names and sequence/order, label the basis as `name_order`, and ask the user to confirm. Never use weak recommendations without approval.

Persist approved mapping under `custom_fields.plane.state_mapping` with `approval_status = approved`, `approved_by`, `verified_at`, `recommendation_basis`, and selected state ids/names. If approval is missing, keep `approval_status = pending` and do not use the mapping for create/update/sync.
## Priority Mapping

Map QA values to Plane priority:
- `Critical`, `P0`, or `P0 - Critical` -> `urgent`
- `High`, `P1`, or `P1 - High` -> `high`
- `Medium`, `P2`, or `P2 - Medium` -> `medium`
- `Low`, `P3`, or `P3 - Low` -> `low`
- unknown or not assessed -> `none`

## Description HTML Structure

Build Plane `description_html` from approved issue package fields:
- summary
- affected test cases
- environment
- reproduction steps
- expected result
- actual result or blocker detail
- evidence refs
- severity normalization reason
- report gaps
- report id and package id
- sign-off recommendation when relevant

Keep HTML simple: paragraphs, headings, ordered/unordered lists, and links.

## Duplicate Handling

Before creating a Plane work item, search with title, affected test case ids, and distinctive error text. If likely duplicate exists, do not create a new work item unless reviewer explicitly overrides. Use `comment_existing` mode when the user wants QA evidence added to an existing Plane item.

## Submission Result

Record successful Plane result in the issue package:
- `submission_status = submitted`
- `submission_result.tool = plane`
- `submission_result.external_issue_id = <work_item_uuid>`
- `submission_result.readable_identifier = ENG-42` when available
- `submission_result.external_issue_url` when available
- `submission_result.submitted_at`
- `submission_result.submitted_by`

If read-back verification fails, set `submission_status = submission_failed` and record the error.

## Troubleshooting

Common Plane MCP failures:
- invalid API key: regenerate token in Plane settings
- missing workspace slug: configure `x-workspace-slug` or `PLANE_WORKSPACE_SLUG`
- wrong workspace slug: check exact Plane URL slug
- insufficient permissions: verify workspace/project role
- resource not found: verify UUID/readable identifier and deletion status
- validation error: check required fields and allowed values
- network error: verify Plane base URL and connectivity

