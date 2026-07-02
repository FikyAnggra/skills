# Changelog

## 0.2.15 - 2026-07-02

- Added `Automation Implemented` as an allowed automation status between `To Be Automate` and `Already Automate`.
- Clarified `Automation Implemented` means automation exists but is not reviewed, approved, or proven as stable maintained coverage.
- Clarified `Already Automate` requires reviewed, approved, maintained automation coverage.
- Updated planning state schema, handoff contract schema, templates, examples, Plane comment summary, and README prompts.

## 0.2.14 - 2026-07-02

- Added Plane parent/sub-work-item planning mode with parent-first intake, sub-work-item id deduplication, per-item state gates, and parent-child context tracking.
- Added required readable-id scenario prefixes for Plane-derived parent and sub-work-item test cases.
- Added managed result comments for every processed Plane sub-work-item while keeping parent-level batch summary comments and parent-level Notion artifacts.
- Added Plane sub-work-item read/create/update/state/delete guidance with explicit delete safeguards and MCP gap records.
- Extended Plane source, output policy, and planning state schemas with sub-work-item context, processed item tracking, sub-work-item comments, and operation policies.

## 0.2.13 - 2026-06-26

- Split generated test cases into `ui_test_cases` and `api_test_cases` while keeping legacy `test_cases` for compatibility.
- Added API test case field model: TC ID, Scenario, Summary, Method, URL, Header, Params, Body, Expected Result, Actual Result, Test Case Status, Automation Status, Notes.
- Added Notion policy and fallback guidance for separate UI and API test case databases.
- Added `templates/test-case-api.md` and downstream handoff refs for UI/API test case lists.

## 0.2.12 - 2026-06-26

- Added explicit Plane page/wiki read and update workflow for QA pages.
- Added Plane page MCP tool mapping for retrieving, creating, updating, and versioned fallback behavior.
- Extended Plane output policy schema with page read-before-update, managed-section update, page tool aliases, and page read/update gap handling.

## 0.2.11 - 2026-06-26

- Changed the Notion display column name from `Pre-Conditions` to `PreConditions`.
- Added Notion view-order policy so the default view, `All Cases`, and every qa-planner-created view use the requested test case column order.
- Added schema fields for Notion view property order, view-order verification, and view-order gap handling.

## 0.2.10 - 2026-06-25

- Updated Notion test case database display columns to use `Pre-Conditions` and `Test Step` in the required order.
- Changed Plane non-`Todo Test` state handling to require separate user confirmation after state lookup; direct item mentions such as `DKI-213` no longer count as approval to proceed.
- Updated Plane output policy schema to use `ask_user_for_confirmation_after_state_read`.

## 0.2.9 - 2026-06-25

- Added Plane QA state workflow for `Backlog Test Human`, `Todo Test`, `Analyze Test`, `Update Test`, `Need Review Test`, and `Ready to Test`.
- Made `Todo Test` the default eligible Plane state and required explicit request/confirmation for other states.
- Added routing rules for analysis-only, update, human review, and ready-to-test transitions.
- Added machine-readable Plane QA state workflow policy in `plane-output-policy.schema.json`.

## 0.2.8 - 2026-06-16

- Enforced exact Notion test case database display column order: TC ID, Scenario, Summary, Test Type, Priority, Pre Conditions, Test Steps, Test Data, Expected Result, Actual Result, Test Case Status, Automation Status, Notes.
- Added logical field mapping from Notion display columns to qa-planner internal test case fields.

## 0.2.7 - 2026-06-15

- Updated Notion test case database rules to use exact `templates/test-case.md` columns, including long fields as database properties.
- Added Notion fallback behavior that creates a page with the same test case table when database creation or schema update is unavailable.
- Clarified Notion test plan page rendering must follow `templates/test-plan.md` with clean Notion page structure.

## 0.2.6 - 2026-06-15

- Added conditional Notion MCP reference for creating QA test plan pages and mandatory test case databases.
- Added Notion output policy schema for page/database creation, database property requirements, views, link capture, idempotency, and fallback behavior.
- Extended planning state and handoff contracts with Notion page/database URLs and IDs.
- Updated core workflow, output rules, quality gates, README, and manifests for Notion MCP output.

## 0.2.5 - 2026-06-13

- Added concrete Plane MCP tool mapping for readable id resolution, work item payloads, comments, activities, relations, cycle/module scope, links, states, members, custom properties, and work item types.
- Added Plane planning-state enrichment for project members, states, work item properties/types, relations, and batch scope.
- Added deterministic Plane write-back guidance for managed comments, HTML pages, external links, dynamic status movement, and optional labels/fields.
- Added Plane handoff tracking guidance for creating QA execution tracker work items and adding them to QA cycles.
- Extended Plane schemas and handoff contracts for MCP context, external links, execution tracker refs, and handoff tracking policy.

## 0.2.4 - 2026-06-13

- Added inline `planning_state.json` fallback skeleton to make state generation deterministic without schema files.
- Added explicit automation status decision rules.
- Added controlled `test_type` values and updated `planning-state.schema.json` to enforce them.
- Added blocking versus non-blocking ambiguity rules.
- Added inline coverage map format and status values.
- Added inline handoff contract fallback shape for executor, automation, and reporter.
- Added large-input batching guidance for large PRDs and high test case counts.

## 0.2.3 - 2026-06-13

- Reduced Plane bias in `SKILL.md` by moving detailed Plane behavior into `references/plane-hybrid.md`, loaded only for Plane inputs.
- Reworked workflow into explicit standard and Plane paths.
- Replaced invalid priority remapping with an `open_questions` clarification rule for any unsupported priority value.
- Split quality gates into content, Plane sync, and status transition groups.
- Unified review behavior into one review loop that supports OK, NOK, and partial feedback.
- Added explicit output decision rules and resource fallback instructions.
- Added source conflict handling and clearer `scenario` versus `summary` examples.

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
