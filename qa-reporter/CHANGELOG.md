# Changelog

## 0.7.3 - 2026-06-26

- Added approval blocking-info guard: if unresolved blockers such as missing env, email, OTP, test data, access, or evidence remain, approve/OK now requires one extra explicit confirmation before approval is processed.
- Applied the guard to Plane `Need Review Issue Report`, Plane `Need Review Report`, Notion review comments, issue creation, issue submission, and Plane state movement.
- Added review-history recording guidance for approval attempts blocked by unresolved questions or required testing information.

## 0.7.2 - 2026-06-25

- Updated Plane `Need Review Issue Report` approval handling: approve/OK now creates approved bug/issue sub work items in `Backlog Issue` for Plane-sourced reports.
- Added post-approval source transition: after created bug/issue sub work items are verified, move the source work item to `Backlog Test`.
- Added failure guard: do not move the source work item to `Backlog Test` when issue sub work item creation or read-back verification fails.

## 0.7.1 - 2026-06-25

- Tightened Plane `Need Issue Report` transition order: move and verify `Generating Issue Report` before creating issue report artifacts.
- Tightened issue-report completion order: move and verify `Need Review Issue Report` before announcing the report is ready for review.
- Added transition failure behavior: stop, keep artifacts draft/review-pending, record blocker in `plane_state_routing`, and report the failed move.
## 0.7.0 - 2026-06-25

- Added Plane-context trigger behavior for Plane, Plane.so, Plane cards/work items, readable ids such as `DKI-179`, Plane URLs, and Plane UUIDs.
- Added Plane state router with automatic starts only from `Need Issue Report` and `Ready to Report`.
- Added issue-report workflow states: `Need Issue Report`, `Generating Issue Report`, `Need Review Issue Report`, and `Backlog Issue`.
- Added testing-report workflow states: `Ready to Report`, `Generating Report`, `Need Review Report`, and `Release Approval`.
- Added guardrail confirmation prompt for Plane work items outside supported QA Reporter workflow states.
- Added `plane_state_routing` to report state schema, template, and example.
- Added `references/plane-state-routing.md`.
## 0.6.0 - 2026-06-18

- Added external approval surface rule: when Notion or Plane is configured, publish/update Notion or Plane review surface before requesting approval.
- Added Notion and Plane external-review references for OK/NOK review instead of local-only approval.
- Added Plane bug label resolution workflow: use existing `bug` label, create it when missing and supported, then apply it to bug work items/sub work items.
- Added bug label resolution fields to issue submission schema and templates.
- Updated SIT template to follow `docs/Template SIT DKI.md` structure with qa-reporter template variables.
- Updated UAT template to follow `docs/Template UAT DKI.md` structure with qa-reporter template variables.
## 0.5.0 - 2026-06-18

- Added Plane source work item comment summary behavior when reading Plane requirements or Notion links from Plane.
- Added idempotent Plane description update behavior for Notion links discovered from source work items.
- Added source-based bug creation strategy: create sub work item when issue comes from a Plane requirement/source work item, otherwise create a normal work item.
- Added `sub_work_item` submission mode, work item creation strategy, issue kind, and bug marker schema fields.
- Added bug/issue marker rules: prefer Bug type, apply bug/issue labels when available, fallback to `[Bug]` or `[Issue]` title prefix.
- Updated Plane reference, issue template, Plane summary comment template, and examples.
## 0.4.0 - 2026-06-17

- Added Plane work item source intake for readable IDs such as `ENG-42`.
- Added Notion link resolver guidance for Notion URLs found in Plane descriptions, comments, and links.
- Added Notion publishing workflow for testing report, SIT, UAT, coverage/risk, issue package, and sign-off outputs.
- Added Plane summary comment workflow with published Notion links.
- Added `source_refs` and `publication_history` to report state schema and templates.
- Added `references/notion-mcp.md`, `templates/notion-report-page.md`, and `templates/plane-summary-comment.md`.
- Updated examples with Plane source work item, Notion planner/executor source refs, Notion published pages, and Plane summary comment history.
## 0.3.0 - 2026-06-17

- Added dynamic Plane state discovery and recommendation workflow.
- Added user approval gate before using any Plane state id for create/update/sync.
- Added `custom_fields.plane.state_mapping` schema with available states, recommendation basis, approval status, and per-intent state selections.
- Removed example use of fixed Plane state names as default behavior; examples now show pending user-approved mapping.
- Updated issue templates to display fetched Plane states and recommended mapping for review.
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








