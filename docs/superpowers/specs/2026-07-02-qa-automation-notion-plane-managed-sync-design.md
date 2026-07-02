# QA-Automation Notion + Plane Managed Sync Design

Date: 2026-07-02
Status: Approved brainstorming design draft for user review

Source artifacts:
- `qa-automation/skills/qa-automation/SKILL.md`
- `qa-planner/skills/qa-planner/SKILL.md`
- `qa-planner/skills/qa-planner/references/notion-mcp.md`
- `qa-planner/skills/qa-planner/references/plane-hybrid.md`
- `qa-executor/skills/qa-executor/SKILL.md`
- `qa-executor/skills/qa-executor/schemas/notion-sync-record.schema.json`
- `qa-executor/skills/qa-executor/schemas/plane-sync-record.schema.json`
- `qa-executor/skills/qa-executor/templates/notion-plane-bridge.json`
- Notion MCP supported tools: `https://developers.notion.com/guides/mcp/mcp-supported-tools`
- Plane MCP server tools: `https://developers.plane.so/dev-tools/mcp-server-tools`

## 1. Goal

Update `qa-automation` so it can use Notion MCP and Plane.so MCP as managed source, workflow, and output surfaces for automation implementation.

The update must follow the existing QA agent pattern:
- keep canonical state in a local JSON artifact;
- treat Notion and Plane as synchronized views/workflow systems;
- use managed sync records and idempotency markers;
- gate destructive actions;
- keep platform-specific details in references loaded only when needed.

For `qa-automation`, the canonical state remains `automation_change_set.json`.

## 2. Scope

In scope:
- Read Notion pages, databases, data sources, comments, users, and MCP payloads when available.
- Create and update Notion pages, rows/pages inside databases, comments, and managed automation output when policy allows.
- Update Notion test case automation status after automation implementation and after validation/review.
- Read Plane work items/cards, sub-work-items, comments, activities, links, relations, states, members, worklogs, cycles/modules, pages/wiki, and MCP payloads when available.
- Create and update Plane comments, description summaries, links, worklogs, pages/wiki, work item fields, and state when policy allows.
- Support Plane/Notion CRUD capability in the skill model while making delete/archive require explicit approval.
- Support Notion + Plane bridge when Plane carries Notion links or both systems are provided.
- Add schemas, templates, references, examples, README, CHANGELOG, and metadata updates.
- Search for relevant external skills after spec approval, then install only with user approval if a trustworthy match exists.

Out of scope:
- Creating test cases from raw requirements.
- Running manual QA as `qa-executor`.
- Writing final SIT/UAT/reporter reports.
- Automatically creating follow-up items for automation gaps.
- Submitting final defects.
- Remote git push, deploy, or merge without explicit approval.
- Deleting or archiving Notion/Plane objects without explicit approval.

## 3. Source Of Truth

`automation_change_set.json` stays canonical.

Notion and Plane are synchronized surfaces:
- Notion is the preferred test case/status source when a Notion test case database/page is provided or discovered.
- Plane is the preferred workflow surface when a Plane work item/card/sub-work-item is provided or discovered.
- When both are active, the bridge stores source refs, sync direction, cross-links, idempotency keys, and partial sync errors.

`qa-automation` must not claim a remote Notion/Plane write succeeded unless the MCP/API/tool response confirms it. If a tool is missing or fails, the local change set remains valid and the sync gap is recorded.

## 4. Automation Status Model

`qa-automation` must align with the updated planner automation status enum:
- `Manual Only`
- `To Be Automate`
- `Automation Implemented`
- `Already Automate`

Status transition rules:
- `To Be Automate` -> `Automation Implemented` when a script is created or updated and source sync is allowed.
- `Automation Implemented` -> `Already Automate` only when script review is `OK` and relevant validation has `passed`.
- Failed validation keeps the case at `Automation Implemented` or previous status and records validation failure.
- Blocking gaps such as missing selector, fixture, data, environment, API contract, repo, or framework keep the case at `To Be Automate` or previous status and record the gap.
- `Manual Only` is unchanged unless a human explicitly changes the automation decision.

Promotion review remains separate. It gates push/deploy/merge readiness, not the transition from `Automation Implemented` to `Already Automate`.

## 5. Notion Managed Sync

### 5.1 Activation

Activate the Notion path when the input or requested output contains:
- Notion page URL;
- Notion database/data source URL;
- Notion page id, database id, or data source id;
- Notion MCP payload;
- request to update automation status in Notion;
- Plane text or links containing a Notion reference that can be classified.

### 5.2 Notion Capabilities

When tools are available, `qa-automation` may:
- search/fetch Notion sources;
- query data sources or database views;
- read comments and users;
- create pages;
- update pages;
- create/update rows as pages in a database/data source;
- create comments;
- create/update a managed automation page only when requested or policy enables it;
- delete/archive Notion objects only after explicit approval.

Tool names may vary by MCP client. The skill should use capability-based language and record `notion_mcp_gap` when a required capability is missing.

### 5.3 Write Policy

Default Notion policy:
- `source_status_update = true` when a writable test case source is identified.
- `comment_sync = true` for concise automation status/result comments when comments are supported or requested.
- `automation_page_sync = false` unless user requests detailed automation output in Notion.
- `result_database_sync = false` unless user requests a dedicated automation/result database.
- `schema_update = false` unless user asks or policy explicitly enables schema changes.
- `delete_requires_confirmation = true`.

Create/update is allowed when the request and write policy are clear. Delete/archive always requires explicit approval after the target is read and the impact is summarized.

### 5.4 Status And Field Updates

For each Notion-sourced test case, preserve source locators:
- database id or data source id;
- row/page id;
- property map;
- URL;
- source role.

Update behavior:
- Set `Automation Status = Automation Implemented` after script create/update.
- Set `Automation Status = Already Automate` only after script review `OK` and validation `passed`.
- Update optional properties only if they exist or schema update is allowed:
  - `Script Path`
  - `Validation Status`
  - `Review Status`
  - `Automation Change Set`
  - `Automation Notes`
  - `Last Automation Sync`
- If required properties are missing and schema update is unavailable or disabled, record `notion_schema_gap` and use a concise comment or `Notes` fallback when possible.

### 5.5 Idempotency

Before creating Notion output, search/fetch for existing managed artifacts using:
- `change_set_id`;
- package id;
- TC ID;
- source Notion page/database id;
- managed marker.

Managed marker:

```text
qa-automation-managed
```

Notion idempotency key shape:

```text
qa-automation:<change_set_id>:notion:<target_type>:<target_id>:<tc_id?>
```

Sync outcomes are recorded in `automation_change_set.notion_sync`.

## 6. Plane Managed Sync

### 6.1 Activation

Activate the Plane path when the input or requested output contains:
- Plane work item/card URL;
- readable id such as `DKI-179`;
- Plane UUID;
- Plane MCP payload;
- work item/card/sub-work-item/comment/page/wiki mention;
- request to comment, update description, change state, update worklog, create/update Plane work item, or sync automation output to Plane.

### 6.2 Plane Capabilities

When tools are available, `qa-automation` may:
- resolve readable ids to work items;
- read work item title, description, comments, activities, links, relations, states, members, worklogs, cycles/modules, and pages/wiki;
- create/update managed comments;
- create/update managed description summary blocks;
- create/update links;
- create/update worklogs;
- create/update pages/wiki when requested;
- create/update work items only when requested as part of automation management;
- update state only when a valid dynamic mapping exists;
- delete/archive work items, sub-work-items, comments, or pages only after explicit approval.

Tool names may vary by MCP client. The skill should use capability-based language and record `plane_mcp_gap` when a required capability is missing.

### 6.3 Dynamic State Policy

There is no fixed automation workflow yet.

Rules:
- Read/list project states before any Plane state movement.
- Use explicit user-provided mapping or work item/package policy when available.
- If no mapping exists, record `plane_state_mapping_gap` and skip state movement.
- If a target state does not exist in the project, record `plane_state_transition_gap` and skip state movement.
- State movement is not required for local automation completion unless the user or package policy says it is.

Default state sync:
- `status_sync = false` unless state mapping is provided/requested.

### 6.4 Write Policy

Default Plane policy:
- `comment_sync = true` when Plane context is active and automation reaches a terminal outcome.
- `description_sync = true` when requested or an existing managed description block is found.
- `status_sync = false` unless state mapping is provided/requested.
- `worklog_sync = false` unless requested.
- `link_sync = true` for script refs, change set refs, review refs, validation refs, and Notion refs when tools allow.
- `page_sync = false` unless requested.
- `create_followup_items = false`.
- `delete_requires_confirmation = true`.

Create/update is allowed when request and write policy are clear. Delete/archive always requires explicit approval after the target is read and impact is summarized.

### 6.5 Plane Outputs

Terminal automation comment should include:
- change set id;
- readable work item id;
- automation status summary;
- TC IDs;
- script paths;
- validation status;
- script review status;
- gaps;
- Notion links when bridge is active;
- next action.

Managed description summary should be concise and bounded by markers, preserving user content outside the managed section.

Managed marker:

```text
qa-automation-managed
```

Plane idempotency key shape:

```text
qa-automation:<change_set_id>:plane:<plane_work_item_id>:<target_type>
```

Sync outcomes are recorded in `automation_change_set.plane_sync`.

## 7. Notion + Plane Bridge

Activate the bridge when:
- a Plane work item contains Notion links;
- the user provides Plane and Notion references in one request;
- a planner/executor handoff includes both refs;
- the user asks to update Notion and Plane together.

Bridge rules:
- `automation_change_set.json` is source of truth.
- Plane is workflow surface.
- Notion is test case/status source and optional detailed automation surface.
- If exactly one high-confidence Notion test case source is discovered from Plane, use it automatically.
- If multiple Notion refs exist or classification is unclear, ask the user to choose.
- Do not duplicate long automation output in Plane when Notion has a detailed automation page. Plane should contain summary and links.
- Add Notion links to Plane comments/description/links when policy allows.
- Add Plane links to Notion comments/page/row when policy allows.
- Partial bridge sync does not invalidate local automation output.

Bridge idempotency key:

```text
qa-automation:<change_set_id>:plane:<plane_ref>:notion:<notion_ref>
```

Bridge sync statuses:
- `not_started`
- `synced`
- `partial`
- `failed`
- `skipped`

## 8. Schema Updates

Update `automation-change-set.schema.json`:
- add `Automation Implemented` to automation status enums;
- add `notion_context`;
- add `notion_write_policy`;
- add `notion_sync`;
- add `plane_context`;
- add `plane_write_policy`;
- add `plane_state_mapping`;
- add `plane_sync`;
- add `notion_plane_bridge`;
- add CRUD operation tracking;
- add delete approval fields;
- add sync gaps and MCP gaps;
- keep platform-specific extensions under `custom_fields` when not core.

Update `automation-status-update.schema.json`:
- support `Automation Implemented`;
- support Notion row/page update outcomes;
- support Plane sync refs;
- support partial/failed/skipped sync result.

Add schemas:
- `notion-sync-record.schema.json`
- `plane-sync-record.schema.json`

Schema enum alignment must be checked against `qa-planner` status values.

## 9. Templates

Add Notion templates:
- `notion-sync-record.json`
- `notion-automation-comment.md`
- `notion-automation-page.md` only if detailed page output is enabled/requested

Add Plane templates:
- `plane-sync-record.json`
- `plane-automation-comment.md`
- `plane-description-summary.md`

Add bridge template:
- `notion-plane-bridge.json`

Existing templates should be updated only when needed:
- `automation-change-set.json`
- `automation-review.md`
- `validation-report.md`
- `patch-summary.md`
- `promotion-checklist.md`

## 10. References

Add references loaded conditionally:
- `references/notion-mcp.md`
- `references/plane-mcp.md`
- `references/notion-plane-bridge.md`

`SKILL.md` should mention these files only in the resource list and conditional intake rules. Detailed MCP tool mapping belongs in references, not the core skill body.

## 11. Examples

Add examples:
- `notion-automation-source.example.json`: Notion test case source with automation status update.
- `plane-automation-work-item.example.json`: Plane work item automation workflow.
- `notion-plane-bridge.example.json`: Plane work item discovering Notion test case source.
- `notion-status-update.example.json`: `To Be Automate -> Automation Implemented -> Already Automate`.
- `plane-automation-comment.example.md`: terminal Plane automation comment.

Examples must show:
- no automatic follow-up item creation for gaps;
- dynamic Plane state mapping behavior;
- delete approval requirement;
- sync gaps when a tool/capability is missing.

## 12. Skill And Package Updates

Update `qa-automation/skills/qa-automation/SKILL.md`:
- frontmatter triggers mention Notion and Plane;
- input intake includes Notion and Plane paths;
- workflow includes source resolution, write policy resolution, sync, and bridge;
- output rules include Notion and Plane;
- quality gates include sync records and redaction;
- resource list includes new schemas/templates/references/examples.

Update:
- `qa-automation/README.md`
- `qa-automation/CHANGELOG.md`
- `qa-automation/.codex-plugin/plugin.json`
- `qa-automation/.claude-plugin/plugin.json`
- `qa-automation/skills/qa-automation/agents/openai.yaml`

## 13. Safety Rules

Write safety:
- Create/update is allowed when request/policy is clear and tool is available.
- Delete/archive in Notion or Plane requires explicit approval after reading target and summarizing impact.
- Remote git push/deploy/merge requires explicit approval.
- If write partially fails, preserve local canonical state and record partial sync.

Redaction:
- Redact tokens, passwords, cookies, credentials, private keys, secrets, authorization headers, session ids, PII, and sensitive logs before Notion/Plane writes.
- If redaction cannot be verified for a screenshot or evidence artifact, keep it local and record an evidence/sync gap.

No auto follow-up creation:
- `create_followup_items = false` by default.
- Blocking gaps are recorded in `automation_change_set.json` and synced as comments/description/status only when policy allows.

## 14. Validation

Implementation validation must run:
- required file structure check;
- JSON parse for all schemas/templates/examples;
- placeholder scan;
- skill frontmatter validation;
- package manifest JSON parse;
- OpenAI metadata shape check;
- stale enum search for automation status values;
- official skill/plugin validators when dependencies are available.

Known validation risk:
- Previous local validator runs failed when Python lacked `yaml`. If this recurs, document dependency failure and run equivalent manual validation.

## 15. Relevant Skill Discovery

After this spec is reviewed and approved, use:
- `find-skills` to search for relevant external skills around MCP, Notion, Plane, JSON schema, or workflow sync.
- `skill-installer` only if a trustworthy relevant skill is found and the user approves installation.

Selection criteria:
- prefer official or high-reputation sources;
- prefer strong install count or credible repository;
- do not install unknown low-trust skills automatically.

If no relevant skill is found, proceed with local implementation using official MCP docs and existing `qa-planner`/`qa-executor` patterns.

## 16. Implementation Notes

Recommended implementation approach:
1. Update schemas.
2. Add templates.
3. Add references.
4. Add examples.
5. Update `SKILL.md`.
6. Update README, CHANGELOG, manifests, and metadata.
7. Validate.

The implementation must remain platform-neutral. Codex, Claude, Notion, and Plane are runtime surfaces, not hard dependencies for the core automation workflow.

## 17. Done Criteria

Design is ready for implementation when:
- user approves this spec;
- scope covers Notion read/create/update/delete policy;
- scope covers Plane read/create/update/delete policy;
- automation status enum includes `Automation Implemented`;
- `Already Automate` transition requires script review OK and validation passed;
- dynamic Plane state handling is explicit;
- no automatic follow-up item creation is included;
- schemas/templates/references/examples are listed;
- validation expectations are clear.
