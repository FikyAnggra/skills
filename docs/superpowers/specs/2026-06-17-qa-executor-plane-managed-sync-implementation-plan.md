# QA Executor Plane Managed Sync Implementation Plan

Date: 2026-06-17
Spec: `docs/superpowers/specs/2026-06-17-qa-executor-plane-managed-sync-design.md`
Status: Draft for implementation approval

## Goal

Update the portable `qa-executor` skill package with Managed Plane QA Sync support.

The implementation must let `qa-executor` accept Plane work items/cards as execution input, require preflight status mapping and write policy before Plane writes, sync execution output through managed Plane comments/status/worklogs/links, and support opt-in Plane wiki/page updates. `execution_result.json` remains the canonical source of truth.

## Target Structure Changes

```text
qa-executor/
  skills/
    qa-executor/
      SKILL.md
      schemas/
        execution-package.schema.json
        execution-result.schema.json
        plane-sync-record.schema.json
      templates/
        plane-execution-comment.md
        plane-wiki-page.md
        plane-sync-record.json
      examples/
        plane-execution-package.example.json
        plane-execution-result.example.json
        plane-execution-comment.example.md
  README.md
  CHANGELOG.md
```

Existing files stay intact unless updated by this plan.

## Work Items

### 1. Update `SKILL.md` for Plane Input and Managed Sync

Add a Plane input path and managed sync workflow.

Content must include:
- Plane triggers: readable id, URL, UUID, MCP payload, work item/card mentions
- Plane intake fields: work item, comments, state, labels, assignees, cycle/module, relations, links, worklogs
- executability rules for Plane input
- `source_not_executable` behavior when Plane work item only has requirements
- preflight status mapping requirement
- Plane write policy requirement
- managed comment behavior
- status transition rules
- worklog rules
- link/relation sync rules
- wiki/page opt-in behavior
- command parsing rules
- idempotency and redaction rules
- Plane quality gates

Acceptance criteria:
- `qa-executor` remains execution-only.
- Skill does not tell executor to create test plans from raw Plane requirements.
- Skill does not submit final defects by default.
- Skill requires status mapping before execution when Plane status sync is requested.
- Wiki/page writes are explicitly opt-in only.

### 2. Update `execution-package.schema.json`

Add optional typed Plane sections.

Fields:
- `plane_context`
  - workspace/project/work item identifiers
  - readable identifier and URL
  - state, assignees, labels
  - cycle/module refs
  - relations
  - source comments
  - attachments or links
- `plane_write_policy`
  - `comment_sync`
  - `status_sync`
  - `worklog_sync`
  - `link_sync`
  - `wiki_sync`
  - `create_followup_items`
  - `requires_confirmation`
- `plane_status_mapping`
  - `on_start`
  - `on_all_passed`
  - `on_any_failed`
  - `on_blocked`
  - `on_partial`
  - `on_cancelled`

Acceptance criteria:
- JSON remains valid.
- Existing required fields and current examples remain valid.
- New fields are optional so non-Plane executor use remains unchanged.
- `wiki_sync` and `create_followup_items` default behavior is documented as false in examples/templates, even if JSON Schema cannot apply defaults at runtime.

### 3. Update `execution-result.schema.json`

Add optional `plane_sync` result section.

Fields:
- `source_work_item`
- `managed_comment_id`
- `wiki_page_id`
- `worklog_id`
- `status_before`
- `status_after`
- `status_transition_reason`
- `linked_refs`
- `sync_status`
- `sync_errors`
- `idempotency_key`

Allowed `sync_status` values:
- `not_started`
- `synced`
- `partial`
- `failed`
- `skipped`

Acceptance criteria:
- JSON remains valid.
- Current non-Plane examples remain valid.
- Plane result example validates structurally with new section.
- Partial Plane sync can be represented without invalidating completed execution results.

### 4. Add `plane-sync-record.schema.json`

Create a schema for idempotent Plane write records.

Record fields:
- `sync_record_id`
- `execution_id`
- `plane_work_item_id`
- `readable_identifier`
- `target_type`
- `target_id`
- `action`
- `idempotency_key`
- `content_hash`
- `sync_status`
- `last_synced_at`
- `errors`
- `custom_fields`

Allowed target types:
- `comment`
- `status`
- `worklog`
- `link`
- `wiki`
- `followup_item`

Allowed sync statuses:
- `not_started`
- `synced`
- `partial`
- `failed`
- `skipped`

Acceptance criteria:
- Schema is valid JSON.
- It supports one record per Plane write action.
- It can prevent duplicate comments/worklogs/wiki/follow-up items by storing the idempotency key and content hash.

### 5. Add Plane Templates

Create:
- `templates/plane-execution-comment.md`
- `templates/plane-wiki-page.md`
- `templates/plane-sync-record.json`

Template requirements:
- comment template includes managed marker `<!-- qa-executor:execution:<execution_id>:<plane_work_item_id> -->`
- comment template includes summary counts, status mapping, per-test table, blockers, retries, issue candidates, evidence refs, reporter handoff ref, automation signal ref, redaction status, review status
- wiki template includes execution summary, test table, blockers, retries, evidence refs, issue candidates, handoff refs, review status
- sync record template matches `plane-sync-record.schema.json`

Acceptance criteria:
- Templates contain no unresolved placeholder markers.
- Templates are safe for Plane comments/pages and avoid raw secret dumps.
- Wiki template text says it is opt-in only.

### 6. Add Plane Examples

Create:
- `examples/plane-execution-package.example.json`
- `examples/plane-execution-result.example.json`
- `examples/plane-execution-comment.example.md`

Example requirements:
- include a Plane readable id such as `ENG-42`
- include status mapping and write policy
- show `wiki_sync: false`
- show managed comment sync
- show status transition after a failed or blocked run
- show optional worklog skipped or synced with reason
- show one issue candidate ref
- show reporter handoff and automation signal refs
- show idempotency key

Acceptance criteria:
- All JSON examples parse.
- Examples demonstrate the selected Managed Plane QA Sync approach.
- Examples do not imply final defect submission by default.

### 7. Update README and CHANGELOG

Update `README.md` with:
- Plane work item/card input support
- preflight status mapping
- write policy and managed sync outputs
- wiki/page opt-in behavior
- idempotency summary
- Plane safety limits

Update `CHANGELOG.md` with a new version entry for Plane Managed Sync.

Acceptance criteria:
- README remains short but complete.
- Changelog records schema, template, example, and behavior additions.
- No platform-specific assumption is required for non-Plane use.

### 8. Optional Plugin Metadata Refresh

Review `.codex-plugin/plugin.json`, `.claude-plugin/plugin.json`, and `agents/openai.yaml`.

Update only if current descriptions no longer represent Plane input/sync capability.

Acceptance criteria:
- Plugin manifests remain valid JSON.
- Metadata does not overpromise final defect submission or final reporting.
- No unsupported manifest fields are added.

### 9. Validate Package

Run local checks:
- required Plane files exist
- all JSON files parse
- `SKILL.md` frontmatter still has `name` and `description`
- placeholder scan across `qa-executor`
- schema enum spot check for new Plane statuses and target types
- plugin manifests parse
- official skill/plugin validators if dependencies allow

Known prior validation issue:
- system `python.exe` previously failed with a Windows logon-session error
- bundled Python previously lacked `yaml`, blocking official validator scripts

Acceptance criteria:
- No invalid JSON.
- No unresolved placeholder markers.
- All new files are referenced by `SKILL.md` or README.
- Official validator failures caused by missing runtime dependencies are documented.

## Implementation Order

1. Update `SKILL.md` with Plane intake, preflight, sync, safety, and quality gates.
2. Update `execution-package.schema.json` with optional Plane input/write-policy/status-mapping fields.
3. Update `execution-result.schema.json` with optional `plane_sync` fields.
4. Add `plane-sync-record.schema.json`.
5. Add Plane templates.
6. Add Plane examples.
7. Update README and CHANGELOG.
8. Refresh plugin metadata only if needed.
9. Run validation checks.
10. Summarize created/changed files and validation results.

## Risks

- Plane MCP tool names or payload shapes may vary by runtime. Mitigation: keep skill instructions tool-agnostic and store unresolved tool gaps in `plane_sync.sync_errors`.
- Status names differ per Plane project. Mitigation: require preflight status mapping and skip missing states with a recorded gap.
- Plane writes may duplicate comments/worklogs on rerun. Mitigation: managed marker, idempotency key, sync record, and content hash.
- Plane input may be raw requirement text, not executable test cases. Mitigation: record `source_not_executable` and route to planner.
- Evidence may contain secrets. Mitigation: redact before Plane write and store refs instead of raw blobs.
- Wiki output could become noisy. Mitigation: `wiki_sync` remains opt-in only.

## Done Criteria

Implementation is done when:
- `qa-executor` accepts Plane work items/cards as documented input.
- Plane preflight status mapping and write policy are documented in `SKILL.md`.
- Schemas support `plane_context`, `plane_write_policy`, `plane_status_mapping`, `plane_sync`, and `plane-sync-record`.
- Plane templates and examples exist.
- README and CHANGELOG describe Plane Managed Sync.
- JSON files parse.
- Placeholder scan is clean.
- Final summary lists validation results and any blocked official validators.
