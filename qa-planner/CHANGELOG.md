# Changelog

## 0.2.2 - 2026-06-12

- Restored mandatory status-move clarification for Plane requirement input when no status movement instruction is provided.
- Updated Plane output policy schema to use `ask_user_before_running` for missing status movement instruction.
- Documented that answering no continues planning with transition status `not_requested`.

## 0.2.1 - 2026-06-12

- Added temporary no-default-status-move behavior; superseded by `0.2.2`, which requires asking the status-move clarification before running Plane requirement input when unspecified.
- Strengthened Plane output behavior so generated planning output must be added or updated on the source Plane card/work item when write tools are available.
- Added attachment-content intake guidance for readable Plane card/work item attachments.
- Added no-assumption rule: ask the user when missing information affects planning accuracy instead of inventing details.
- Extended Plane source and output policy schemas for attachment extraction and missing status instruction behavior.

## 0.2.0 - 2026-06-12

- Added Plane hybrid integration guidance for MCP assistant and Plane native agent workflows.
- Added Plane source, output policy, and sync record schemas.
- Added Plane comment and wiki/page templates.
- Added Plane work item input, sync record, and output comment examples.
- Updated planning state schema with optional Plane refs, artifact outputs, output policy, transition policy, and sync records.
- Updated handoff contract schema with optional Plane context and artifact refs.
- Added full sync behavior: comment, attachments, wiki/page update, page link, and optional status movement after successful sync.
- Added status transition rules: all source statuses are allowed, target status can be configured, and missing target status skips movement without failing planning.
- Added Plane safety and idempotency rules for redaction, managed sections, checksums, and duplicate prevention.

## 0.1.0 - 2026-06-11

- Added portable `qa-planner` skill package.
- Added Codex and Claude plugin wrappers.
- Added `planning-state`, `handoff-contract`, and `review-feedback` JSON Schemas using draft 2020-12.
- Added Markdown test plan and test case templates.
- Copied `Template Test Case.xlsx` into the package templates directory without content changes.
- Added handoff contract template for `qa-executor`, `qa-automation`, and `qa-reporter`.
- Added sample requirement, planning state, and rendered test case examples.
- Compatibility target: portable skill folder plus Codex and Claude-style plugin wrappers.

## Future Migration Notes

- Keep `planning_state` as the canonical source of truth when adding new output formats.
- Add platform-specific wrappers without changing core `SKILL.md` behavior.
- Extend schemas through `custom_fields`, `tags`, and refs before introducing breaking schema changes.
