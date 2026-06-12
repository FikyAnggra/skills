# Changelog

## 0.2.0 - 2026-06-12

- Added Plane hybrid integration guidance for MCP assistant and Plane native agent workflows.
- Added Plane source, output policy, and sync record schemas.
- Added Plane comment and wiki/page templates.
- Added Plane work item input, sync record, and output comment examples.
- Updated planning state schema with optional Plane refs, artifact outputs, output policy, transition policy, and sync records.
- Updated handoff contract schema with optional Plane context and artifact refs.
- Added full sync behavior: comment, attachments, wiki/page update, page link, and optional status movement after successful sync.
- Added status transition rules: all source statuses are allowed, ask before running when target status is missing, and skip status movement when target status is not found.
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
