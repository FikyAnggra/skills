# QA-Planner Plane Hybrid Implementation Plan

Date: 2026-06-12
Spec: `docs/superpowers/specs/2026-06-12-qa-planner-plane-hybrid-design.md`
Status: Draft for implementation approval

## Goal

Update the portable `qa-planner` package with Plane hybrid integration. The skill must accept Plane work item/card inputs, generate QA planning artifacts, sync outputs back to Plane when tools are available, update wiki/page content when useful, and optionally move the source work item/card status after successful sync.

The implementation must preserve existing `qa-planner` behavior for non-Plane usage.

## Target Files

Create:

```text
qa-planner/skills/qa-planner/schemas/plane-source.schema.json
qa-planner/skills/qa-planner/schemas/plane-output-policy.schema.json
qa-planner/skills/qa-planner/schemas/plane-sync-record.schema.json
qa-planner/skills/qa-planner/templates/plane-comment.md
qa-planner/skills/qa-planner/templates/plane-page.md
qa-planner/skills/qa-planner/examples/plane-work-item-input.json
qa-planner/skills/qa-planner/examples/plane-sync-record.example.json
qa-planner/skills/qa-planner/examples/plane-output-comment.example.md
```

Update:

```text
qa-planner/skills/qa-planner/SKILL.md
qa-planner/skills/qa-planner/schemas/planning-state.schema.json
qa-planner/skills/qa-planner/schemas/handoff-contract.schema.json
qa-planner/README.md
qa-planner/CHANGELOG.md
qa-planner/.codex-plugin/plugin.json
qa-planner/.claude-plugin/plugin.json
qa-planner/skills/qa-planner/agents/openai.yaml
```

## Work Items

### 1. Add Plane Source Schema

Create `plane-source.schema.json`.

Schema must cover:
- workspace slug/id
- project id and readable identifier
- work item/card UUID and readable identifier
- source type: work item, comment, attachment, page, module, cycle, payload
- source URL
- title, description, comment refs, attachment refs, page refs
- status name/id as metadata only
- parent/child/linked work item refs

Acceptance criteria:
- Valid JSON Schema draft 2020-12.
- Does not restrict input by source status.
- Allows unknown Plane/custom fields through `custom_fields`.

### 2. Add Plane Output Policy Schema

Create `plane-output-policy.schema.json`.

Schema must cover:
- write mode enum: `read_only`, `comment_only`, `attach_artifacts`, `wiki_update`, `full_sync`
- artifact format preferences: JSON, Markdown, Excel, PDF, Word
- wiki/page rules
- comment behavior
- attachment behavior
- status transition policy
- behavior when Plane tool is unavailable

Acceptance criteria:
- Default behavior is represented as `full_sync` for Plane inputs.
- Transition target can be absent before user answer.
- Target status not found behavior is `skip_status_move`.

### 3. Add Plane Sync Record Schema

Create `plane-sync-record.schema.json`.

Schema must cover:
- package id, source agent, external id
- source work item refs
- managed comment refs
- attachment refs
- wiki/page refs
- page link refs
- artifact checksums and versions
- sync status
- transition status
- gap/error notes

Acceptance criteria:
- Supports idempotent reruns.
- Status enum includes `full_sync_succeeded`, `full_sync_failed`, `status_transition_succeeded`, `status_transition_skipped`.
- Can record `plane_sync_gap` and target status missing reasons.

### 4. Update Planning State Schema

Update `planning-state.schema.json`.

Add:
- `plane_refs` support for source inputs or top-level integration metadata
- `artifact_outputs[]`
- `plane_output_policy`
- `plane_transition_policy`
- `plane_sync_records[]`

Acceptance criteria:
- Existing example still can be adapted without breaking core planning sections.
- Existing priority, test case status, and automation status enums remain unchanged.
- Plane fields are optional for non-Plane usage.

### 5. Update Handoff Contract Schema

Update `handoff-contract.schema.json`.

Add optional Plane metadata:
- source work item UUID/readable id
- project/workspace refs
- artifact refs
- page/comment/attachment refs
- downstream Plane context

Acceptance criteria:
- Existing `qa-executor`, `qa-automation`, and `qa-reporter` contracts remain valid.
- Plane refs are optional.

### 6. Add Plane Templates

Create `plane-comment.md`.

Include:
- package id/version
- planning status
- test case counts
- coverage gaps/open questions
- artifact links
- wiki/page link
- status transition result
- reviewer next action

Create `plane-page.md`.

Include sections:
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

Acceptance criteria:
- Plane comment is concise and not a full duplicate of the plan.
- Plane page has clear managed sections for safe update behavior.

### 7. Add Plane Examples

Create examples:
- `plane-work-item-input.json`: sample Plane work item/card input.
- `plane-sync-record.example.json`: sample successful full sync with status transition.
- `plane-output-comment.example.md`: sample concise Plane comment.

Acceptance criteria:
- Example includes readable id and UUID refs.
- Example demonstrates source status as metadata only.
- Example demonstrates status transition after successful sync.
- Example demonstrates target status missing skip behavior or gap field.

### 8. Update SKILL.md

Add sections:
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

Also update frontmatter description to mention Plane work item/card input, comments, attachments, wiki/page updates, full sync, and status movement.

Acceptance criteria:
- Skill states all Plane source statuses are allowed.
- Skill requires asking status transition question before running when Plane input lacks status-move instruction.
- Skill moves status only after successful Plane sync.
- Skill skips status move when target status is missing.
- Skill keeps non-Plane behavior intact.

### 9. Update README and Changelog

Update `README.md` with:
- Plane hybrid integration overview
- supported Plane inputs
- full sync behavior
- status transition behavior
- fallback behavior

Update `CHANGELOG.md` with a new `0.2.0` entry.

Acceptance criteria:
- README remains platform-neutral.
- Changelog records schemas/templates/examples and behavior change.

### 10. Update Plugin Metadata

Update `.codex-plugin/plugin.json`, `.claude-plugin/plugin.json`, and `agents/openai.yaml` metadata to mention Plane hybrid planning.

Acceptance criteria:
- Plugin manifests remain valid.
- Capability wording remains accurate and concise.

### 11. Validate

Run checks:
- JSON parse for all manifest, schema, template JSON, and example JSON files.
- Skill quick validation.
- Plugin validation.
- Placeholder scan for `TODO`, `TBD`, `[TODO`, `PLACEHOLDER`.
- Confirm all files referenced in `SKILL.md` and README exist.

Acceptance criteria:
- No invalid JSON.
- No unresolved placeholder markers.
- Existing enums are unchanged.
- `Template Test Case.xlsx` remains untouched.

## Implementation Order

1. Add new Plane schemas.
2. Update existing planning and handoff schemas.
3. Add Plane templates.
4. Add Plane examples.
5. Update `SKILL.md`.
6. Update README, CHANGELOG, and plugin metadata.
7. Validate JSON and skill/plugin metadata.
8. Summarize generated files and validation results.

## Risks and Mitigations

- Plane MCP tool names may differ by runtime. Mitigation: document capability-based behavior and fallback, not one fixed tool name.
- Full sync may duplicate output if refs are missing. Mitigation: add sync record and checksum-based idempotency.
- Wiki updates may overwrite user content. Mitigation: mark managed sections and preserve manual sections.
- Status move may conflict with team workflow. Mitigation: ask when target status is absent and only move after successful sync.
- Sensitive data may leak into Plane. Mitigation: add redaction gate before comment/wiki/attachment sync.

## Done Criteria

Implementation is done when:

- New Plane schemas, templates, and examples exist.
- Existing schemas support optional Plane refs and sync state.
- `SKILL.md` documents Plane hybrid behavior end to end.
- README and changelog describe Plane integration.
- Plugin metadata mentions Plane support.
- JSON files parse.
- Plugin and skill validation pass, or any validator environment limitation is clearly documented.
- No unresolved placeholder markers remain.
- Final summary lists created/updated files and validation results.
