# Plane MCP Reference

Load this file only when the request includes Plane work items/cards, readable ids such as `DKI-179`, Plane URLs, UUIDs, MCP payloads, comments, links, pages/wiki, cycle/module scope, or requests to sync automation status to Plane.

## Tool Capabilities

Use Plane MCP tools when available. Tool names vary by client; use the available equivalent and record `plane_mcp_gap` when a required capability is missing.

Read capabilities:
- resolve readable ids to work item payloads
- retrieve work item/card/sub-work-item by id
- list comments, activities, links, relations, labels, assignees, states, worklogs, cycles, modules, and pages/wiki
- read project/workspace pages used as QA context

Write capabilities:
- create/update managed comments
- create/update managed description summary blocks
- create/update links
- create/update worklogs when requested
- create/update pages/wiki when requested
- create/update work item fields when requested
- change state only when dynamic mapping is available and valid
- create/update work items only when requested as part of automation management
- delete/archive only after explicit user approval

## Activation

Plane path is active when any of these exist:
- Plane work item/card URL
- readable id such as `DKI-179`
- Plane UUID
- Plane MCP payload
- work item/card/sub-work-item/comment/page/wiki mention
- request to comment, update description, change state, update worklog, create/update Plane work item, or sync automation output to Plane

## Dynamic State Mapping

There is no fixed automation workflow yet.

Rules:
- Read/list project states before any state movement.
- Use user-provided mapping or work item/package policy if available.
- If no mapping exists, record `plane_state_mapping_gap` and skip state movement.
- If target state does not exist, record `plane_state_transition_gap` and skip state movement.
- State movement is not required for local automation completion unless the user or package policy says it is.

Do not invent states.

## Write Policy

Default policy:
- `comment_sync = true` when Plane context is active and automation reaches a terminal outcome
- `description_sync = true` when requested or an existing managed block is found
- `status_sync = false` unless state mapping is provided/requested
- `worklog_sync = false` unless requested
- `link_sync = true` for script refs, change set refs, review refs, validation refs, and Notion refs when tools allow
- `page_sync = false` unless requested
- `create_followup_items = false`
- `delete_requires_confirmation = true`

Create/update is allowed when request and policy are clear. Delete/archive requires explicit approval after reading the target and summarizing impact.

Do not automatically create Plane follow-up items for gaps. Record gaps in `automation_change_set.json` and sync them as comments/description/status only when policy allows.

## Managed Outputs

Terminal comment should include:
- change set id
- readable work item id
- automation status summary
- TC IDs
- script paths
- validation status
- script review status
- gaps
- Notion links when bridge is active
- next action

Description summary should stay concise and be bounded by managed markers. Preserve all user content outside managed sections.

Managed marker:

```text
qa-automation-managed
```

Idempotency key:

```text
qa-automation:<change_set_id>:plane:<plane_work_item_id>:<target_type>
```

Before creating comments/pages/links/worklogs, read existing Plane content when tools allow and update the managed artifact if present.

## Redaction And Fallback

Before writing to Plane, redact secrets, tokens, passwords, cookies, credentials, authorization values, session ids, PII, and sensitive logs.

If Plane tools are missing or fail:
- keep `automation_change_set.json` complete
- generate local Markdown/JSON artifacts
- record `plane_mcp_gap`, `plane_read_gap`, `plane_state_mapping_gap`, or `plane_sync_gap`
- do not claim remote sync succeeded
