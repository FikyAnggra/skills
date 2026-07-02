# Changelog

## 0.2.0 - 2026-07-02

- Added Notion managed sync support for automation status updates, comments, optional automation pages, sync records, and source locators.
- Added Plane managed sync support for workflow comments, description summaries, dynamic state mapping, links, worklogs/pages policy, and sync records.
- Added Notion + Plane bridge support for discovered Notion test case sources from Plane and cross-link sync.
- Added `Automation Implemented` as an automation status between `To Be Automate` and `Already Automate`.
- Updated status rules so `Already Automate` requires script review `OK` and relevant validation `passed`.
- Added delete/archive approval policy for Notion and Plane.
- Added Notion/Plane schemas, templates, references, and examples.
- Recorded that gap follow-up items are not created automatically.

## 0.1.0 - 2026-06-11

- Added portable `qa-automation` package with Codex and Claude plugin wrappers.
- Added canonical `qa-automation` skill instructions.
- Added schemas for automation change sets, reviews, validation results, and automation status updates.
- Added templates for change sets, reviews, validation reports, patch summaries, and promotion checklists.
- Added references for Playwright TypeScript fallback, locator policy, and branch policy.
- Added examples for planner contracts, executor automation signals, change sets, and automation review output.
- Compatibility targets: Codex plugin loader and Claude-compatible project workflows.
- Future migration notes: keep schema additions backward-compatible through `custom_fields` unless a versioned breaking change is required.
