# Changelog

## 0.2.0 - 2026-06-17

- Added optional Plane MCP adapter guidance for approved issue submission packages.
- Added `references/plane-mcp.md` with Plane transport, auth, identifier, field mapping, duplicate handling, and troubleshooting notes.
- Extended issue submission schema with `target_tracker`, `submission_mode`, duplicate check metadata, Plane routing fields under `custom_fields.plane`, and read-back verification fields.
- Extended report submission history schema to allow Plane results.
- Updated issue and bug report templates with Plane routing and duplicate check sections.
- Updated examples to show Plane tracker routing without automatic submission.
## 0.1.0 - 2026-06-11

- Added initial portable `qa-reporter` package.
- Added Codex and Claude-compatible plugin manifests.
- Added canonical `qa-reporter` skill instructions.
- Added schemas for report state, issue submission package, reporter review, and sign-off recommendation.
- Added templates for report state, test report, issue package, bug report, SIT document, UAT document, and coverage-risk summary.
- Added examples for manual execution, exploratory issues, canonical report state, rendered test report, and rendered issue package.
- Compatibility target: portable agent package with Codex plugin metadata and Claude-compatible wrapper metadata.
- Future migration note: keep schema changes versioned and preserve `custom_fields` for platform-specific extensions.

