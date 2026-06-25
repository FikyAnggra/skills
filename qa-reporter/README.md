# QA Reporter

QA Reporter is a portable agent skill package for turning QA planner data, executor results, manual testing notes, or exploratory issue findings into governed reporting packages.

## Contents

- Codex plugin wrapper in `.codex-plugin/plugin.json`
- Claude-compatible plugin wrapper in `.claude-plugin/plugin.json`
- Portable skill in `skills/qa-reporter/SKILL.md`
- JSON schemas for report state, issue submission packages, reporter reviews, and sign-off recommendations
- Markdown and JSON templates for test reports, issue packages, bug reports, SIT/UAT documents, coverage-risk summaries, and report state
- Examples for manual execution, exploratory issues, report state, and rendered outputs

## Input Paths

- Planner plus executor: consume `planning_state`, `execution_result`, issue candidates, coverage map, risk analysis, evidence refs, and approval metadata.
- Manual execution result: normalize human test execution statuses and notes into `report_state.json`.
- Exploratory issue finding: convert a defect observation into a bug report and issue submission package, with gaps called out when evidence is incomplete.

## Outputs

Default outputs are `report_state.json`, `test-report.md`, `issue-package.md`, `bug-report.md`, `sit-document.md`, `uat-document.md`, and `coverage-risk-summary.md`. Other formats should render from `report_state.json` instead of becoming separate sources of truth.

## Sign-Off Model

Recommendations are `Go`, `Conditional Go`, `No-Go`, or `Not Assessed`. The recommendation must show its rule and reason. Human override is allowed when reviewer, reason, timestamp, previous recommendation, and final recommendation are recorded.

## External Approval Review

When approval is required and Notion or Plane is used, QA Reporter publishes or updates the Notion/Plane review surface first, then asks reviewers to approve from that external surface. Local-only approval is used only if the user explicitly accepts fallback after Notion/Plane publishing is blocked.

## Review and Submission Gate

Reports use multi-level `OK`/`NOK` review. `NOK` feedback updates `report_state.json` and re-renders impacted artifacts. `OK` approves the package.

Issue submission is never automatic. Submission requires explicit user request, approved issue package, available tool/config, complete required fields, duplicate handling when relevant, and redaction status `passed`.

## Plane MCP Adapter

QA Reporter includes an optional Plane adapter for approved issue packages. It can prepare Plane work item payloads, resolve project routing data, run duplicate checks, create a work item or intake item when tools are available, comment on an existing work item, attach evidence links, and record read-back verification in `submission_history`. Plane-specific IDs live under `custom_fields.plane` so the core package stays portable.

### Plane State Routing

When input contains Plane context such as `Plane`, `Plane.so`, a work item/card, readable id like `DKI-179`, or a Plane URL/UUID, QA Reporter reads the Plane work item first and routes by current state. Automatic reporting starts only from `Need Issue Report` or `Ready to Report`. Other states require user confirmation before report generation or state movement.

Issue-report route: `Need Issue Report` -> `Generating Issue Report` -> `Need Review Issue Report`; approval `OK` creates bug/issue work items in `Backlog Issue`.

Testing-report route: `Ready to Report` -> `Generating Report` -> `Need Review Report`; approval `OK` moves the source work item to `Release Approval`.

### Plane Source Sync and Bug Creation Rules

When QA Reporter reads Notion links from a Plane source work item, it records a Plane comment summary and updates the work item description with a QA Reporter links section, unless read-only mode is requested. When a requirement/source Plane work item produces a bug, QA Reporter creates a sub work item under that source work item after approval. If no source Plane work item exists, it creates a normal work item. Created work items must be visibly marked as bugs by type, label, or title prefix fallback.

### Dynamic Plane State Mapping

QA Reporter does not hardcode Plane states. Before creating or syncing Plane work items, it must fetch/list states for the target project, show every available state to the user, recommend a mapping to QA intents, and ask the user to approve or edit the mapping. Approved mapping is stored under `custom_fields.plane.state_mapping` per project.

## Plane Work Item and Notion Flow

QA Reporter can use a Plane work item such as `ENG-42` as a reporting anchor. It reads the work item, extracts Notion links from description/comments/links, fetches linked Notion planner and executor artifacts, builds `report_state.json`, publishes testing/SIT/UAT reports back to Notion, and comments a compact summary with Notion links back to Plane when requested.

This is reporting and traceability only. Test case design still belongs to `qa-planner`, and test execution still belongs to `qa-executor`.

Supported linked-source flow:

- `qa-planner` creates test cases/report in Notion and links them in Plane.
- `qa-executor` updates Notion test cases/report and links evidence in Plane.
- `qa-reporter` reads Plane and Notion sources, creates final testing/SIT/UAT reports in Notion, creates bug work items/sub work items in Plane after state mapping approval, and comments summary links back to Plane.

## Downstream Workflow

`qa-planner` creates planning state and reporter handoff data. `qa-executor` produces execution results, evidence refs, issue candidates, and reporter handoff data. `qa-reporter` creates the governed reporting package and optional approved issue submission package.
