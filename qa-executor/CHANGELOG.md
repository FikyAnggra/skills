# Changelog

## 0.1.0 - 2026-06-11

- Added initial portable `qa-executor` package.
- Added Codex and Claude plugin wrappers.
- Added platform-neutral `qa-executor` skill instructions.
- Added execution package, execution result, execution review, and issue candidate schemas.
- Added result, report, issue candidate, reporter handoff, and automation signal templates.
- Added manual input and execution result examples.
- Compatibility target: Codex skills, Claude-compatible skill wrappers, and platform-neutral agent runtimes.
- Migration note: keep `execution_result.json` as source of truth when future schemas add fields.
