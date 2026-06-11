# QA Planner

`qa-planner` is a portable QA planning package for agent platforms. It converts requirements, PRDs, user stories, API specs, UI flows, screenshots, database notes, and constraints into governed planning artifacts.

## Contents

- `.codex-plugin/plugin.json`: Codex plugin wrapper.
- `.claude-plugin/plugin.json`: Claude-compatible metadata wrapper.
- `skills/qa-planner/SKILL.md`: canonical portable planner instructions.
- `skills/qa-planner/schemas/`: JSON Schemas for planning state, handoff contracts, and review feedback.
- `skills/qa-planner/templates/`: Markdown, JSON, and Excel templates for QA planning outputs.
- `skills/qa-planner/examples/`: sample requirement, planning state, and rendered test cases.
- `skills/qa-planner/agents/openai.yaml`: Codex UI metadata.

## Use With Agent Platforms

Install or load this folder as a plugin when supported. The canonical instruction source is `skills/qa-planner/SKILL.md`; platform wrappers only point agents to that skill and its resources.

For platforms without plugin support, copy `skills/qa-planner/` as a skill folder and keep `schemas/`, `templates/`, and `examples/` beside `SKILL.md`.

## Inputs

Provide requirements, PRDs, acceptance criteria, UI or API notes, screenshots, technical constraints, or existing QA artifacts. For revision, provide human review feedback with `OK` or `NOK` and the target area.

## Outputs

The source of truth is `planning_state.json`. Human-readable plans, test cases, coverage maps, spreadsheet outputs, and downstream handoff contracts are rendered from that state.

Expected outputs include:
- test plan
- test cases using the `Template Test Case.xlsx` field model
- requirement coverage map
- `qa-executor`, `qa-automation`, and `qa-reporter` handoff contracts
- review history and changelog entries

## Review Loop

`OK` review sets `planning_status` to `approved`. `NOK` review records structured feedback, updates impacted planning state sections, and re-renders impacted artifacts until review passes.

## Downstream Handoff

- `qa-executor`: receives selected approved test cases, run policy, environment, test data, and coverage refs.
- `qa-automation`: receives automation candidates, existing automation updates, manual-only cases, and automation gaps.
- `qa-reporter`: receives planning summary, coverage refs, risk refs, planned scope, exclusions, and approval metadata.

`qa-planner` does not execute tests, create defect tickets, write final automation scripts, or produce final execution reports.
