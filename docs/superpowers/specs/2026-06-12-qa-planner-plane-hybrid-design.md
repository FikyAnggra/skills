# QA-Planner Plane Hybrid Integration Design

Date: 2026-06-12
Status: Draft for user review
Base package: `qa-planner/`
Related docs:
- `docs/superpowers/specs/2026-06-10-qa-planner-design.md`
- `docs/superpowers/specs/2026-06-10-qa-planner-implementation-plan.md`

## 1. Purpose

Update `qa-planner` so it can work powerfully with Plane.so while staying portable across agent platforms.

The update adds a Plane hybrid integration layer. `qa-planner` can accept requirements from Plane work items/cards and can sync planning artifacts back to the related Plane item through comments, attachments, and wiki/page updates when Plane tools are available. If Plane tools are not available, the skill falls back to local or chat-delivered Markdown/JSON artifacts.

The core planning model remains unchanged: `planning_state` is the source of truth, and `qa-planner` still owns planning only.

## 2. Goals

- Accept Plane work item/card requirements as first-class input.
- Support both MCP assistant workflow and Plane native agent workflow.
- Attach or link planning outputs back to the correct Plane work item/card.
- Create or update Plane wiki/page content when useful.
- Allow status movement after successful Plane sync, with user confirmation when the target status was not provided.
- Preserve traceability from Plane source to requirements, test cases, coverage, artifacts, and downstream handoff contracts.
- Avoid duplicate comments, attachments, and pages across repeated runs.
- Keep the skill platform-neutral and safe when Plane tools are unavailable.

## 3. Non-Goals

- Do not run tests.
- Do not create defect tickets.
- Do not produce final test execution reports.
- Do not write final automation scripts.
- Do not update execution statuses without verified executor results or explicit user instruction.
- Do not require Plane for normal `qa-planner` usage.
- Do not overwrite manually written Plane wiki content aggressively.

## 4. Operating Modes

### 4.1 Portable Mode

Use this mode when Plane tools are not available or the user provides non-Plane inputs. `qa-planner` behaves as it does today: it generates `planning_state`, test plans, test cases, coverage maps, review history, and downstream handoff contracts.

### 4.2 Plane MCP Assistant Mode

Use this mode when the user provides a Plane readable id, UUID, URL, or payload and MCP/API-style tools are available. The agent resolves Plane references, reads the work item/card context, generates planning artifacts, and syncs output back to Plane according to policy.

### 4.3 Plane Native Agent Mode

Use this mode when the planner is invoked inside Plane, for example through an @mention in a work item comment. The agent uses the provided Plane context, writes progress and final response to the related work item/card, and follows the same output and transition policies.

## 5. Plane Input Sources

`qa-planner` must accept requirements from these Plane sources:

- work item/card description
- work item/card title
- comments and review replies
- attachments
- project pages or wiki pages
- parent/child work items
- linked or referenced work items
- module context
- cycle/sprint context
- direct Plane payloads from MCP/API tools

Inputs are normalized into `planning_state.source_inputs[]`. Plane-specific references are stored under `plane_refs` so artifacts can trace back to their source.

Example `plane_refs` shape:

```json
{
  "plane_refs": {
    "workspace_slug": "acme",
    "project_id": "project-uuid",
    "project_identifier": "ENG",
    "work_item_id": "work-item-uuid",
    "readable_identifier": "ENG-42",
    "source_type": "work_item",
    "source_url": "https://app.plane.so/acme/projects/ENG/issues/42",
    "comment_ids": [],
    "attachment_ids": [],
    "page_ids": []
  }
}
```

## 6. Plane Source Status Policy

There is no source status restriction.

`qa-planner` may use a Plane work item/card from any status as requirement input when the user provides that item as input. This includes `Backlog`, `Todo`, `In Progress`, `Done`, `Cancelled`, archived/custom statuses, and any team-specific status.

The skill must not block planning based on source status. Source status is metadata only.

## 7. Plane Output Policy

Default Plane output mode is `full_sync`.

Supported write modes:

- `read_only`: generate artifacts but do not write to Plane.
- `comment_only`: write a concise summary comment to the source work item/card.
- `attach_artifacts`: write summary comment and attach generated artifacts.
- `wiki_update`: create/update Plane wiki/page content and link it to the work item/card when supported.
- `full_sync`: comment, attach artifacts, create/update wiki/page when needed, link wiki/page to work item/card, and then perform configured status transition after successful sync.

When the user does not provide a mode, use `full_sync` for Plane inputs.

## 8. Artifact Output Strategy

When Plane sync is enabled, `qa-planner` should generate all artifact formats available in the runtime:

- JSON: `planning_state.json`, handoff contracts, sync record
- Markdown: test plan, test cases, coverage map, Plane comment/page content
- Excel: test cases based on `Template Test Case.xlsx`
- PDF/Word: generated only when the platform or toolchain supports them

Minimum fallback output is JSON plus Markdown.

The artifact set must be recorded in `planning_state.artifact_outputs[]` with path/ref, format, checksum when available, and Plane attachment/page/comment references after sync.

## 9. Plane Comment Behavior

The work item/card comment should be short and operational. It should not duplicate the full plan when attachments or wiki pages exist.

Comment content should include:

- planning package id and version
- source work item/card reference
- planning status
- test case count by priority and automation status
- coverage gaps or open questions
- artifact links/attachment refs
- wiki/page link when created or updated
- status transition result, if attempted
- next action for reviewer

If there is an existing managed comment from `qa-planner`, update or append a revision note according to tool capability. Do not spam duplicate final comments for the same package/version.

## 10. Plane Wiki/Page Behavior

`qa-planner` may create or update Plane wiki/page content.

Wiki/page should be created or updated when at least one condition is true:

- output is too long for a work item comment
- test case inventory is large
- the plan is reusable across multiple work items, modules, cycles, or regression suites
- user explicitly asks for wiki/page update
- planning output should become QA knowledge
- planning is approved or review-ready and needs long-lived storage

Wiki/page should not be created or updated when:

- user chooses `read_only`, `comment_only`, or disables wiki update
- Plane write permission or page capability is unavailable
- critical secrets, credentials, tokens, or PII have not been redacted
- the output is small enough for comment/attachment and no reusable value exists

Default page title format:

```text
QA Plan - <PROJECT>-<SEQ> - <feature title>
```

Default page sections:

- Summary
- Source Work Item
- Scope
- Assumptions and Open Questions
- Risk Analysis
- Test Strategy
- Test Cases
- Coverage Map
- Downstream Handoffs
- Review History
- Artifact Links
- Changelog

When updating an existing page, `qa-planner` must update only its managed sections. It must preserve manual notes and non-managed user content. If content conflicts are detected, record a planning gap or open question instead of overwriting aggressively.

## 11. Plane Status Transition Policy

`qa-planner` may move the source Plane work item/card status.

If the user provides Plane input but does not specify whether to move status after planning, the agent must ask before running the planning workflow.

Default question:

```text
Setelah QA planning selesai dan output berhasil diattach/update ke Plane, apakah work item perlu dipindahkan status? Jika ya, ke status apa?
```

User responses:

- If the user says no, run planning without status transition.
- If the user says yes and provides a target status, store that target in `plane_transition_policy` and run planning.
- If the user gives unclear status instruction, ask one concise follow-up before running.

Transition timing:

- Move status only after planning artifacts are generated and Plane sync succeeds.
- Plane sync success means the required configured output actions completed: comment, attachments, wiki/page when applicable, and link when applicable.

Skip transition when:

- target status is not found in the Plane project
- Plane sync fails
- transition tool/API is unavailable
- user chose no status move

If target status is not found, still complete planning and Plane output sync. Record `status_transition_skipped` in the sync record and include a short reason in the Plane comment.

Status movement is not QA approval. `planning_status = approved` remains controlled by the OK/NOK review loop.

Example transition policy:

```json
{
  "plane_transition_policy": {
    "enabled": true,
    "target_status": "QA Ready",
    "move_timing": "after_successful_plane_sync",
    "requires_user_confirmation": true,
    "requested_by": "user",
    "status_before": "In Progress",
    "status_after": "QA Ready",
    "on_target_status_not_found": "skip_status_move"
  }
}
```

## 12. Review Loop From Plane

Plane comments can drive review updates.

Review interpretation:

- `OK`: mark planning package as approved, record review metadata, and update managed Plane outputs.
- `NOK`: capture feedback in `review_history`, set `planning_status = revision_required`, revise impacted sections, and update managed Plane outputs.

Useful review commands:

- `OK`
- `NOK: <feedback>`
- `scope only`
- `regenerate cases`
- `update wiki`
- `attach excel`
- `move to <status>`

The skill should treat ambiguous review comments as normal feedback and ask one concise question only when the ambiguity blocks revision.

## 13. Idempotency and Sync Record

`qa-planner` must avoid duplicate Plane output.

Each sync should track:

- package id
- source agent
- external id
- source work item/card refs
- managed comment refs
- attachment refs
- wiki/page refs
- page link refs when available
- artifact checksums
- artifact version
- sync status
- transition status
- error or gap notes

Run behavior:

- update existing managed page/comment when refs exist and tool capability supports update
- upload new attachments only when content changed or version changed
- append changelog rather than duplicating full content
- record gaps when updates are not possible

Example sync status values:

- `not_started`
- `comment_synced`
- `attachments_synced`
- `wiki_synced`
- `full_sync_succeeded`
- `full_sync_failed`
- `status_transition_succeeded`
- `status_transition_skipped`

## 14. Traceability

Plane traceability must be visible in canonical state and rendered outputs.

Traceability links:

- Plane work item/card to source requirement
- source requirement to test case
- test case to coverage map item
- planning state to artifact outputs
- artifact outputs to Plane attachments/pages/comments
- planning state to downstream handoff contracts
- downstream handoff contracts back to Plane source refs

Each test case should be able to show which Plane input produced it and where its artifact output was synced.

## 15. Data Model Changes

### 15.1 New Schemas

Add these files:

- `skills/qa-planner/schemas/plane-source.schema.json`
- `skills/qa-planner/schemas/plane-output-policy.schema.json`
- `skills/qa-planner/schemas/plane-sync-record.schema.json`

### 15.2 Existing Schema Updates

Update `planning-state.schema.json` to include:

- `plane_refs` on source input records or a top-level Plane integration section
- `artifact_outputs[]`
- `plane_output_policy`
- `plane_transition_policy`
- `plane_sync_records[]`

Update `handoff-contract.schema.json` to include:

- Plane source refs
- Plane artifact refs
- source work item/card readable id and UUID
- optional downstream Plane context

### 15.3 New Templates

Add these files:

- `skills/qa-planner/templates/plane-comment.md`
- `skills/qa-planner/templates/plane-page.md`

### 15.4 New Examples

Add these files:

- `skills/qa-planner/examples/plane-work-item-input.json`
- `skills/qa-planner/examples/plane-sync-record.example.json`
- optional `skills/qa-planner/examples/plane-output-comment.example.md`

## 16. Skill Instruction Changes

Update `skills/qa-planner/SKILL.md` with these sections:

- Plane hybrid mode trigger and fallback
- Plane input intake rules
- Plane output policy and write modes
- Plane comment behavior
- Plane attachment behavior
- Plane wiki/page behavior
- Plane status transition policy
- Plane review loop
- Plane idempotency and sync record
- Plane traceability rules
- Plane safety rules

The frontmatter description should mention Plane work items/cards, comments, attachments, wiki/page updates, full sync, and status movement.

## 17. Safety and Privacy

Before writing to Plane comment, attachment, or wiki/page, `qa-planner` must check:

- no secrets, tokens, passwords, cookies, API keys, credentials, or private keys are included
- PII is removed or explicitly approved for target destination
- test account details are not exposed in public or broad-access pages
- output destination is appropriate for sensitivity level
- manual wiki content is preserved

If redaction is needed, redact before sync and record a note in `planning_state` or sync record.

If Plane write fails, return local/Markdown/JSON output and record `plane_sync_gap`. Do not move work item status after failed sync.

## 18. Quality Gates

Before final response or Plane sync, verify:

- Plane source refs are resolved as far as available.
- `planning_state` remains the canonical source of truth.
- test case enum values still match the approved values.
- every Plane requirement maps to at least one test case or an exclusion/gap.
- artifacts are generated in all available formats, with JSON and Markdown minimum fallback.
- output policy is known.
- status transition policy is known for Plane input before running.
- Plane sync record is created or updated.
- duplicate output is avoided through idempotency refs/checksums.
- wiki/page update preserves manual content.
- status move happens only after successful Plane sync.
- missing target status skips transition without failing planning.
- secrets and sensitive data are redacted before Plane write.

## 19. User Interaction Rules

When Plane input is used and the user does not mention status transition, ask before running:

```text
Setelah QA planning selesai dan output berhasil diattach/update ke Plane, apakah work item perlu dipindahkan status? Jika ya, ke status apa?
```

Do not ask this question when:

- user already gave a target status
- user explicitly said not to move status
- user selected `read_only`
- no Plane write tool is available and the run is fallback-only

If target status is not found, do not ask again by default. Continue planning, skip transition, and record the skip reason.

## 20. Open Decisions

No unresolved design decisions remain from the current brainstorming session.

Implementation may still need to map exact Plane MCP tool names available in the runtime, because different Plane integrations may expose work item, attachment, page, and status operations differently.

## 21. Acceptance Criteria

The update is complete when:

- `qa-planner` accepts Plane work item/card inputs in its instructions.
- Plane-specific schemas, templates, and examples exist.
- `planning-state.schema.json` supports Plane refs, artifact outputs, output policy, transition policy, and sync records.
- `handoff-contract.schema.json` carries Plane refs to downstream agents.
- `SKILL.md` documents full sync, wiki/page update, idempotency, status movement, and fallback behavior.
- User status transition question is documented.
- Source status is unrestricted.
- Status move runs only after successful Plane sync.
- Missing target status causes skip, not planning failure.
- README and CHANGELOG mention Plane hybrid integration.
- JSON files parse successfully.
- No placeholder markers remain.
