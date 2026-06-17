# QA Executor Plane Managed Sync Design

Date: 2026-06-17
Status: Approved brainstorming design
Target package: `qa-executor`
References:
- `qa-executor/skills/qa-executor/SKILL.md`
- `qa-executor/skills/qa-executor/schemas/execution-package.schema.json`
- `qa-executor/skills/qa-executor/schemas/execution-result.schema.json`
- Plane MCP tool reference: `https://developers.plane.so/dev-tools/mcp-server#tool-reference`

## 1. Goal

Update `qa-executor` so it can use Plane.so work items/cards as execution input and sync execution output back to Plane through a managed, idempotent workflow.

The integration must support Plane work item intake, execution status mapping, managed comments, optional worklogs, optional links/relations, issue candidate references, and opt-in wiki/page updates. `execution_result.json` remains the source of truth. Plane is a synced view and workflow surface, not the canonical execution state.

## 2. Scope

In scope:
- Accept Plane readable ids such as `ENG-42`, Plane URLs, UUIDs, or MCP payloads as executor input.
- Read Plane work item/card context, including description, comments, state, labels, assignees, cycle/module refs, relations, links, and existing worklogs when tools allow.
- Extract executable test cases from Plane content only when minimum executor fields exist.
- Ask for status mapping before execution when not provided.
- Ask for Plane write policy before any Plane write.
- Sync execution progress and final summary through a managed Plane comment.
- Move Plane status using user-approved status mapping and available project states.
- Add one execution worklog when enabled.
- Add evidence, issue candidate, reporter handoff, and automation signal refs in Plane output.
- Create/update Plane wiki/page only when explicitly enabled.
- Maintain idempotency records to avoid duplicate comments, wiki pages, worklogs, and follow-up refs.

Out of scope:
- Creating test plans or test cases from raw requirements. If a Plane work item only contains requirements without executable test cases or expected results, route to `qa-planner` or record `source_not_executable`.
- Producing final reports, SIT, UAT, or sign-off documents. Those remain `qa-reporter` scope.
- Writing final automation scripts. That remains `qa-automation` scope.
- Submitting final defect tickets by default. `qa-executor` may create issue candidates or follow-up work item drafts, and may create Plane follow-ups only when explicitly allowed.

## 3. Plane Workflow

### 3.1 Intake

When the input contains a Plane signal, `qa-executor` resolves the Plane source before execution.

Plane signals include:
- readable id such as `ENG-42`
- Plane work item/card URL
- Plane UUID payload
- user instruction mentioning Plane work item, card, issue, module, cycle, comment, wiki, or page

The executor should read, when tools allow:
- work item id and readable identifier
- project id and project identifier
- current state
- title and description
- comments and recent activity
- labels, assignees, priority, estimate, dates
- cycle and module refs
- parent/child and linked items
- external links and attachment refs
- existing worklogs

If Plane context cannot be read, record a `plane_read_gap` and continue only when another executable source is available.

### 3.2 Executability Rules

Plane input is executable only when one of these is present:
- qa-planner executor handoff embedded or linked from the Plane work item
- test case table with minimum fields: `TC ID`, `Scenario`, `Test Step`, `Expected Result`
- structured execution package JSON
- explicit user-provided test cases in the same request

If the Plane work item contains only requirements, acceptance criteria, or feature text, do not invent test cases. Record `source_not_executable` and ask whether to route the work item to `qa-planner`.

## 4. Preflight Policy

Before execution, `qa-executor` must resolve Plane write behavior.

### 4.1 Status Mapping

Ask the user to map statuses unless a valid mapping is already provided.

Required status mapping keys:
- `on_start`
- `on_all_passed`
- `on_any_failed`
- `on_blocked`
- `on_partial`
- `on_cancelled`

The mapping must be flexible. The executor reads project states, matches requested state names, and skips any missing transition with a recorded gap instead of failing execution.

Example prompt:

```text
Plane status mapping needed before execution. Which project states should qa-executor use for:
- start
- all passed
- any failed
- blocked
- partial
- cancelled
```

### 4.2 Write Policy

Ask or resolve:
- `comment_sync`: create/update managed progress and final summary comment
- `status_sync`: move Plane state using approved mapping
- `worklog_sync`: create/update one execution worklog
- `link_sync`: add links/relations for evidence, issue candidates, handoffs, or follow-ups
- `wiki_sync`: create/update Plane wiki/page; default `false`
- `create_followup_items`: create Plane follow-up items for failed/blocked candidates; default `false`
- `requires_confirmation`: require explicit confirmation for Plane writes; default `true` unless user/session policy says otherwise

Default behavior for the selected design:
- managed comments enabled when user allows Plane write-back
- status sync enabled only after status mapping is resolved
- worklog sync optional
- link sync optional
- wiki sync disabled unless explicitly requested
- follow-up item creation disabled unless explicitly requested

## 5. Execution Flow

1. Resolve Plane source refs and read work item context.
2. Normalize executable input into an execution package.
3. Resolve Plane status mapping and write policy.
4. Resolve run policy: `initial_run`, `full_regression`, `failed_retest`, `smoke`, `targeted`, or `automation_validation`.
5. Resolve execution mode: `api`, `ui`, `db`, `hybrid`, or `manual_instruction`.
6. If configured, move Plane item to `on_start` and create/update the managed progress comment.
7. Execute selected cases or produce manual instructions when tools/access are unavailable.
8. Capture evidence, retries, blockers, issue candidates, reporter handoff, and automation signal.
9. Write or update `execution_result.json` as source of truth.
10. Sync final result to Plane according to write policy.
11. Apply final status transition using approved mapping.
12. Send execution package for human OK/NOK review.

## 6. Plane Sync Behavior

### 6.1 Managed Comments

Use a single managed comment for progress and final result unless the user requests versioned comments.

Managed marker:

```html
<!-- qa-executor:execution:<execution_id>:<plane_work_item_id> -->
```

Progress comment should include:
- execution id
- run mode
- selected count
- current test case when running
- pass/fail/block/untested/retest counts
- started time
- status mapping summary

Final comment should include:
- execution id and package id
- summary counts
- per-test status table
- blockers
- retries
- issue candidate refs
- evidence refs
- reporter handoff ref
- automation signal ref
- redaction status
- review status

### 6.2 Status Transitions

Final status transition decision:
- execution started -> `on_start`
- all selected passed -> `on_all_passed`
- any selected failed -> `on_any_failed`
- any blocked and no failed -> `on_blocked`
- mixed selected/unselected, untested, or incomplete -> `on_partial`
- user cancels run -> `on_cancelled`

If the mapped state does not exist in Plane project states, skip transition and record `plane_sync.status_transition_gap`.

### 6.3 Worklogs

When `worklog_sync = true`, create or update one worklog per `execution_id`.

Worklog description should include:
- execution id
- package id
- selected count
- passed/failed/blocked count
- run mode
- source work item readable id

Do not create duplicate worklogs for the same execution id.

### 6.4 Links and Relations

When `link_sync = true`, sync refs for:
- evidence links
- execution result artifact
- issue candidates
- reporter handoff
- automation signal
- created follow-up work items

If Plane tool support is unavailable for a link/relation type, keep refs in the managed comment and record a sync gap.

### 6.5 Wiki/Page Sync

Wiki/page sync is opt-in only.

Create or update a managed execution evidence page when:
- user explicitly says `publish wiki`, `update wiki`, or equivalent
- or `plane_write_policy.wiki_sync = true`

Wiki/page content should include:
- execution summary
- test table
- blockers
- retries
- evidence refs
- issue candidates
- reporter handoff refs
- automation signal refs
- review status

Do not write raw secrets, raw logs, or unredacted payloads.

## 7. Data Model Changes

### 7.1 Execution Package

Add optional `plane_context`:

```json
{
  "workspace_id": "",
  "project_id": "",
  "project_identifier": "ENG",
  "work_item_id": "",
  "readable_identifier": "ENG-42",
  "url": "",
  "state": "",
  "assignees": [],
  "labels": [],
  "cycle": null,
  "module": null,
  "relations": [],
  "source_comments": [],
  "attachments_or_links": []
}
```

Add optional `plane_write_policy`:

```json
{
  "comment_sync": true,
  "status_sync": true,
  "worklog_sync": false,
  "link_sync": false,
  "wiki_sync": false,
  "create_followup_items": false,
  "requires_confirmation": true
}
```

Add optional `plane_status_mapping`:

```json
{
  "on_start": "In Progress",
  "on_all_passed": "QA Passed",
  "on_any_failed": "QA Failed",
  "on_blocked": "Blocked",
  "on_partial": "In Progress",
  "on_cancelled": "Cancelled"
}
```

### 7.2 Execution Result

Add optional `plane_sync`:

```json
{
  "source_work_item": "ENG-42",
  "managed_comment_id": "",
  "wiki_page_id": null,
  "worklog_id": null,
  "status_before": "Ready for QA",
  "status_after": "QA Failed",
  "status_transition_reason": "At least one selected test failed.",
  "linked_refs": [],
  "sync_status": "synced",
  "sync_errors": [],
  "idempotency_key": "qa-executor:EXEC-20260617-001:ENG-42"
}
```

Allowed `sync_status` values:
- `not_started`
- `synced`
- `partial`
- `failed`
- `skipped`

### 7.3 New Plane Sync Record Schema

Add `plane-sync-record.schema.json` to track write idempotency.

One record per Plane write action:
- target type: `comment`, `status`, `worklog`, `link`, `wiki`, `followup_item`
- target id
- action
- idempotency key
- content hash
- sync status
- last synced timestamp
- errors

## 8. Templates and Examples

Add templates:
- `plane-execution-comment.md`
- `plane-wiki-page.md`
- `plane-sync-record.json`

Add examples:
- `plane-execution-package.example.json`
- `plane-execution-result.example.json`
- `plane-execution-comment.example.md`

## 9. Safety Rules

- Never write to Plane before preflight confirms write policy and status mapping.
- Never move Plane status to a state name that does not exist in the project state list.
- Never create duplicate managed comments; use the managed marker and idempotency key.
- Never create duplicate worklogs for the same execution id.
- Never update wiki/page unless `wiki_sync = true`.
- Never create final defect tickets by default.
- Create Plane follow-up work items only when `create_followup_items = true`.
- Redact token, password, cookie, credential, secret, authorization value, session id, and PII before Plane write.
- Store evidence as refs/links, not raw large evidence blobs.
- If Plane write partially fails, keep `execution_result.json` complete and mark `plane_sync.sync_status = partial`.
- On rerun, update existing managed comment/wiki/sync record unless user asks for a new version.

## 10. Plane Command Parsing

Support read-only command interpretation from Plane comments or user text:
- `run smoke`
- `run TC0001, TC0004`
- `retest failed`
- `publish wiki`
- `sync reporter handoff`

Commands can propose run policy or write policy. They do not bypass preflight confirmation when Plane writes, status movement, wiki update, worklog, link sync, or follow-up creation would happen.

## 11. Cycle and Module Awareness

If the source work item belongs to a Plane cycle or module, carry those refs into `execution-package` and `execution-result`.

The executor may use cycle/module context for reporting and comments. It should not expand execution scope to an entire cycle/module unless linked executable test cases or explicit user scope exists.

## 12. Quality Gates

Before review-ready output:
- Plane source was read or `plane_read_gap` was recorded.
- Plane input was executable or `source_not_executable` was recorded.
- Status mapping was resolved before execution.
- Write policy was resolved before Plane writes.
- Managed comment was synced or sync gap was recorded.
- Status transition succeeded or was skipped with a reason.
- Worklog synced or skipped with a reason.
- Wiki skipped unless explicitly enabled.
- All Plane writes were redacted.
- `execution_result.json` and Plane managed output match.
- Duplicate Plane comments, wiki pages, worklogs, and follow-up items were avoided.

## 13. Implementation Impact

Files expected to change:
- `qa-executor/skills/qa-executor/SKILL.md`
- `qa-executor/skills/qa-executor/schemas/execution-package.schema.json`
- `qa-executor/skills/qa-executor/schemas/execution-result.schema.json`
- `qa-executor/skills/qa-executor/schemas/plane-sync-record.schema.json`
- `qa-executor/skills/qa-executor/templates/plane-execution-comment.md`
- `qa-executor/skills/qa-executor/templates/plane-wiki-page.md`
- `qa-executor/skills/qa-executor/templates/plane-sync-record.json`
- `qa-executor/skills/qa-executor/examples/plane-execution-package.example.json`
- `qa-executor/skills/qa-executor/examples/plane-execution-result.example.json`
- `qa-executor/skills/qa-executor/examples/plane-execution-comment.example.md`
- `qa-executor/README.md`
- `qa-executor/CHANGELOG.md`
- optional plugin metadata descriptions if needed

## 14. Open Decisions

Resolved decisions:
- Use Managed Plane QA Sync, not minimal sync or full command center.
- Wiki/page update is opt-in only.
- Status mapping must be asked before execution when absent.
- Keep `qa-executor` only; do not update `qa-planner` in this design.

No unresolved blocking decisions remain for implementation planning.
