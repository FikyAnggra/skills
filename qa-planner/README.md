# QA Planner

`qa-planner` is a portable QA planning package for agent platforms. It converts requirements, Plane.so work items/cards, PRDs, user stories, API specs, UI flows, screenshots, database notes, and constraints into governed planning artifacts.

## Contents

- `.codex-plugin/plugin.json`: Codex plugin wrapper.
- `.claude-plugin/plugin.json`: Claude-compatible metadata wrapper.
- `skills/qa-planner/SKILL.md`: canonical portable planner instructions.
- `skills/qa-planner/schemas/`: JSON Schemas for planning state, handoff contracts, review feedback, Plane sources, Plane output policy, and Plane sync records.
- `skills/qa-planner/templates/`: Markdown, JSON, Excel, Plane comment, and Plane page templates for QA planning outputs.
- `skills/qa-planner/examples/`: sample requirement, planning state, Plane work item input, Plane sync record, Plane comment, and rendered test cases.
- `skills/qa-planner/agents/openai.yaml`: Codex UI metadata.

## Use With Agent Platforms

Install or load this folder as a plugin when supported. The canonical instruction source is `skills/qa-planner/SKILL.md`; platform wrappers only point agents to that skill and its resources.

For platforms without plugin support, copy `skills/qa-planner/` as a skill folder and keep `schemas/`, `templates/`, and `examples/` beside `SKILL.md`.

## Inputs

Provide requirements, Plane work item/card ids or payloads, PRDs, acceptance criteria, UI or API notes, screenshots, technical constraints, or existing QA artifacts. For revision, provide human review feedback with `OK` or `NOK` and the target area.

Plane inputs can come from work item/card title and description, comments, readable attachment content, wiki/pages, parent or child work items, linked work items, modules, cycles, or direct MCP/API payloads. Plane source status is metadata only; `qa-planner` may plan from any source status when the user provides the requirement input. If required information is missing, the planner asks the user instead of assuming.

## Outputs

The source of truth is `planning_state.json`. Human-readable plans, test cases, coverage maps, spreadsheet outputs, and downstream handoff contracts are rendered from that state.

Expected outputs include:
- test plan
- test cases using the `Template Test Case.xlsx` field model
- requirement coverage map
- `qa-executor`, `qa-automation`, and `qa-reporter` handoff contracts
- Plane comment, attachment, wiki/page, page link, sync record, and optional status transition output when Plane tools are available
- review history and changelog entries

## Plane Hybrid Sync

Default Plane write mode is `full_sync`: add or update a concise comment on the source work item/card, attach generated artifacts to that source item, create or update a wiki/page when useful, link the page to the work item/card, and move status only after sync succeeds when a target status was explicitly configured.

When Plane input is used and the user did not provide a status move instruction, the planner must ask before running:

```text
Setelah QA planning selesai dan output berhasil diattach/update ke Plane, apakah work item perlu dipindahkan status? Jika ya, ke status apa?
```

If the user says no, planning continues without status movement and transition status is `not_requested`.

If the target status is not found, planning and Plane output still complete, status movement is skipped, and the sync record notes `status_transition_skipped`. If Plane tools are unavailable, the planner falls back to JSON and Markdown outputs and records the sync gap.

## Review Loop

`OK` review sets `planning_status` to `approved`. `NOK` review records structured feedback, updates impacted planning state sections, and re-renders impacted artifacts until review passes.

## Downstream Handoff

- `qa-executor`: receives selected approved test cases, run policy, environment, test data, and coverage refs.
- `qa-automation`: receives automation candidates, existing automation updates, manual-only cases, and automation gaps.
- `qa-reporter`: receives planning summary, coverage refs, risk refs, planned scope, exclusions, and approval metadata.

`qa-planner` does not execute tests, create defect tickets, write final automation scripts, or produce final execution reports.
