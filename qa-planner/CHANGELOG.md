# Changelog

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
