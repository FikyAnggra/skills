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

## Review and Submission Gate

Reports use multi-level `OK`/`NOK` review. `NOK` feedback updates `report_state.json` and re-renders impacted artifacts. `OK` approves the package.

Issue submission is never automatic. Submission requires explicit user request, approved issue package, available tool/config, complete required fields, and redaction status `passed`.

## Downstream Workflow

`qa-planner` creates planning state and reporter handoff data. `qa-executor` produces execution results, evidence refs, issue candidates, and reporter handoff data. `qa-reporter` creates the governed reporting package and optional approved issue submission package.
