# QA-Automation Notion + Plane Managed Sync Implementation Plan

Date: 2026-07-02
Spec: `docs/superpowers/specs/2026-07-02-qa-automation-notion-plane-managed-sync-design.md`
Status: Approved for implementation

## Goal

Update the portable `qa-automation` package so it can use Notion MCP and Plane.so MCP as managed source, workflow, and output surfaces while keeping `automation_change_set.json` canonical.

## Work Items

### 1. Search Relevant Skills

Use `find-skills` to search for relevant skills around MCP, Notion, Plane, JSON schema, and workflow sync.

Acceptance criteria:
- Search results are reviewed for trustworthiness.
- No skill is installed without user approval.
- If no useful skill is found, implementation proceeds using official docs and local QA agent patterns.

### 2. Update Change Set And Status Schemas

Update:
- `automation-change-set.schema.json`
- `automation-status-update.schema.json`

Add:
- `notion-sync-record.schema.json`
- `plane-sync-record.schema.json`

Acceptance criteria:
- Automation status enum supports `Manual Only`, `To Be Automate`, `Automation Implemented`, `Already Automate`.
- Change set supports `notion_context`, `notion_write_policy`, `notion_sync`, `plane_context`, `plane_write_policy`, `plane_state_mapping`, `plane_sync`, and `notion_plane_bridge`.
- Delete/archive approval fields exist for Notion and Plane.
- Sync gaps and MCP gaps can be recorded.
- JSON parses.

### 3. Add Templates

Add:
- `notion-sync-record.json`
- `notion-automation-comment.md`
- `notion-automation-page.md`
- `plane-sync-record.json`
- `plane-automation-comment.md`
- `plane-description-summary.md`
- `notion-plane-bridge.json`

Update when needed:
- `automation-change-set.json`
- `automation-review.md`
- `promotion-checklist.md`

Acceptance criteria:
- Templates render from `automation_change_set`.
- Plane output is summary-first and idempotent.
- Notion status update path is explicit.
- Bridge template records cross-links and partial sync.

### 4. Add References

Add:
- `references/notion-mcp.md`
- `references/plane-mcp.md`
- `references/notion-plane-bridge.md`

Acceptance criteria:
- `SKILL.md` stays concise and loads references conditionally.
- Notion reference covers read/create/update/delete policy, status update, idempotency, fallback, and redaction.
- Plane reference covers read/create/update/delete policy, dynamic state mapping, comments, description, links, worklogs, pages/wiki, and redaction.
- Bridge reference covers discovery, classification, cross-linking, duplication policy, and partial sync.

### 5. Add Examples

Add:
- `notion-automation-source.example.json`
- `plane-automation-work-item.example.json`
- `notion-plane-bridge.example.json`
- `notion-status-update.example.json`
- `plane-automation-comment.example.md`

Acceptance criteria:
- Examples show `To Be Automate -> Automation Implemented -> Already Automate`.
- Examples show dynamic Plane state handling.
- Examples show no automatic follow-up item creation for gaps.
- Examples show delete approval requirement.

### 6. Update Skill Instructions

Update `skills/qa-automation/SKILL.md`.

Acceptance criteria:
- Frontmatter triggers mention Notion, Plane, MCP payloads, managed sync, status updates, comments, description summaries, dynamic Plane state, and bridge behavior.
- Core rule still forbids test planning, manual execution, final reports, final defects, remote push/merge/deploy without approval, and delete/archive without approval.
- Input paths include Planner, Executor, manual, Notion, Plane, and bridge.
- Workflow resolves write policy before writes.
- Quality gates include sync records, status update rules, redaction, idempotency, and partial sync handling.

### 7. Update Package Metadata

Update:
- `README.md`
- `CHANGELOG.md`
- `.codex-plugin/plugin.json`
- `.claude-plugin/plugin.json`
- `skills/qa-automation/agents/openai.yaml`

Acceptance criteria:
- Human docs mention Notion/Plane managed sync.
- Plugin metadata matches skill behavior.
- CHANGELOG records versioned update.

### 8. Validate

Run:
- required file structure check;
- JSON parse;
- placeholder scan;
- frontmatter check;
- manifest parse;
- metadata shape check;
- stale enum search;
- official validators if dependencies are available.

Acceptance criteria:
- All JSON parses.
- No unresolved placeholders.
- Status enum is consistent.
- Any validator dependency limitation is documented.

## Done Criteria

Implementation is done when:
- all target schemas/templates/references/examples exist;
- `SKILL.md` and metadata are updated;
- validation passes or dependency-limited validators are documented;
- final summary lists changed files, validation result, and skill search outcome.
