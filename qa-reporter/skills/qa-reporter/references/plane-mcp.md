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

## QA Reporter State Routing

When Plane input is a work item or readable issue id, fetch the work item and route by exact current state before generating output.

Automatic generation is allowed only from:
- `Need Issue Report`
- `Ready to Report`

Supported workflow states:
- `Need Issue Report`: move to `Generating Issue Report` before generating the issue report; verify the state by read-back; generate issue report; then move to `Need Review Issue Report` and verify again before commenting ready for review.
- `Generating Issue Report`: continue or revise issue report, then move to `Need Review Issue Report` and verify the state by read-back.
- `Need Review Issue Report`: for `NOK`, move to `Generating Issue Report`; for `OK`/approve input, first run the Approval Blocking-Info Guard. If unresolved blockers exist, ask a second explicit confirmation before approval, issue creation, or source state movement. After the guard passes, create approved bug/issue sub work items in `Backlog Issue` when the report source is a Plane work item, verify the created items by read-back, then move the source work item to `Backlog Test` and verify the state by read-back. If no Plane source work item exists, create normal bug/issue work items in `Backlog Issue` and skip the source move.
- `Ready to Report`: move to `Generating Report`, generate testing report, then move to `Need Review Report`.
- `Generating Report`: continue or revise testing report, then move to `Need Review Report`.
- `Need Review Report`: for `NOK`, move to `Generating Report`; for `OK`, first run the Approval Blocking-Info Guard. If the guard passes, move to `Release Approval`.

If current state is outside the workflow, ask the user before generating report or moving state. If target states such as `Generating Issue Report`, `Need Review Issue Report`, `Backlog Issue`, `Backlog Test`, `Generating Report`, `Need Review Report`, or `Release Approval` are missing, run state discovery and ask the user to map states.

Store route decisions in `report_state.plane_state_routing`.

## Approval Blocking-Info Guard

Before accepting `OK`, `approve`, or equivalent approval from Plane comments, Notion-linked reviews, or user chat, scan `report_state`, report gaps, issue package gaps, coverage gaps, metric gaps, blocked test cases, and source comments for unresolved data needed to execute, assess, retest, submit, or sign off a test case.

Examples: missing env, email, OTP, account, credential, role, permission, test data, access, evidence, expected result, acceptance rule, open question, open blocking question, blocked reproduction step, or unresolved dependency.

If any blocker exists, ask a second explicit confirmation that lists the blockers:

```text
Masih ada informasi yang memblokir approval: env, email, OTP. Apakah kamu yakin ingin approve tanpa informasi tersebut?
```

Do not create Plane work items, sub work items, submit issues, move the source work item to `Backlog Test`, or move the work item to `Release Approval` until the user either supplies the missing information or gives the second explicit confirmation.

## Source Work Item Sync

When a Plane work item is used as qa-reporter source context, add a compact comment summary to that work item unless the user requested read-only mode. Include source summary, extracted Notion links, classified Notion roles, report gaps, and next reporting action.

When Notion links are read from a Plane work item, update the source work item description with an idempotent `QA Reporter Links` section. Preserve existing description content. If the section already exists, update only that section. Include Notion links and roles. Do not include secrets, raw evidence, tokens, or PII.

## Work Item vs Sub Work Item Creation

If an issue package is derived from a Plane requirement/source work item, create a sub work item under that source work item. This keeps bug/issue visibility attached to the requirement work item. If there is no source Plane work item, create a normal work item in the target project. Use `comment_existing` only when the user chooses to update an existing item instead of creating a new one.

For `Need Review Issue Report` approval, the source Plane work item itself is the parent. Create approved bug/issue candidates as sub work items under that parent in `Backlog Issue`. After every created sub work item is read back and verified, move the parent/source work item to `Backlog Test`. If any sub work item creation or verification fails, do not move the parent/source work item.

## Bug Label Resolution

Before creating a Plane bug work item or sub work item, resolve the `bug` label in the target project.

Workflow:
1. List labels for the target project.
2. If a label named `bug` exists, use that label id.
3. If no `bug` label exists and label creation is available, create a `bug` label and use the created label id.
4. If label creation fails or labels are unsupported, continue only if the work item is still visibly marked by Bug type or `[Bug]`/`[Issue]` title prefix.
5. Record the result in `bug_marker.label_resolution_status` and `bug_marker.labels_applied`.

Local adapters may expose this as `list_labels(project_id)` and `create_label(project_id, label_data)`. Use exact available tool names in the current runtime.
## Bug/Issue Marking

Every created Plane work item or sub work item must be visibly marked as a bug/issue. Prefer work item type `Bug` when available. Resolve or create label `bug`, then apply it. Apply additional labels such as `issue` or `qa-reported` when available. If no type or label can mark the item, prefix the title with `[Bug]` or `[Issue]`. Record the chosen marker in `bug_marker`.
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



