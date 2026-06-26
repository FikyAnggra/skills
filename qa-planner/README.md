# QA Planner

`qa-planner` is a portable QA planning package for agent platforms. It converts requirements, PRDs, user stories, API specs, UI flows, screenshots, database notes, constraints, and optional Plane.so work item/card inputs into governed planning artifacts.

## Contents

- `.codex-plugin/plugin.json`: Codex plugin wrapper.
- `.claude-plugin/plugin.json`: Claude-compatible metadata wrapper.
- `skills/qa-planner/SKILL.md`: canonical portable planner instructions.
- `skills/qa-planner/references/plane-hybrid.md`: conditional Plane integration instructions loaded only for Plane inputs.
- `skills/qa-planner/references/notion-mcp.md`: conditional Notion integration instructions loaded only for Notion output or Notion source inputs.
- `skills/qa-planner/schemas/`: JSON Schemas for planning state, handoff contracts, review feedback, Plane sources, Plane output policy, and Plane sync records.
- `skills/qa-planner/templates/`: Markdown, JSON, Excel, Plane comment, and Plane page templates for QA planning outputs.
- `skills/qa-planner/examples/`: sample requirement, planning state, Plane work item input, Plane sync record, Plane comment, and rendered test cases.
- `skills/qa-planner/agents/openai.yaml`: Codex UI metadata.

## Use With Agent Platforms

Install or load this folder as a plugin when supported. The canonical instruction source is `skills/qa-planner/SKILL.md`; platform wrappers only point agents to that skill and its resources.

For platforms without plugin support, copy `skills/qa-planner/` as a skill folder and keep `schemas/`, `templates/`, and `examples/` beside `SKILL.md`.

## Inputs

Provide requirements, Plane work item/card ids or payloads, PRDs, acceptance criteria, UI or API notes, screenshots, technical constraints, or existing QA artifacts. For revision, provide human review feedback with `OK` or `NOK` and the target area.

Plane inputs can come from work item/card title and description, comments, activities, readable attachment content, wiki/pages, parent or child work items, relations, linked work items, modules, cycles, or direct MCP/API payloads. Plane-specific rules live in `skills/qa-planner/references/plane-hybrid.md` and are loaded only for Plane inputs. If Plane MCP tools are available, the planner resolves readable ids such as `ENG-42`, reads comments, activity history, linked pages/wiki, expands requested cycle/module scope, captures relations and project metadata, and asks the user instead of assuming when required information is missing or conflicting.

Plane QA planning follows a state workflow. By default, `qa-planner` processes Plane work items only from `Todo Test`, moves them to `Analyze Test` for analysis/routing, uses `Update Test` when artifacts must be created or updated, moves completed work to `Need Review Test`, and moves approved work to `Ready to Test`. Any non-`Todo Test` source state requires separate user confirmation after the current state is known. Mentioning a specific item id or asking for direct planning is not confirmation.

Notion output can create or update a QA test plan page and a Notion test case database. The test case artifact must be a Notion database when `notion-create-database` and required `notion-update-data-source` support are available. Database columns and managed table views use the required display order: `TC ID`, `Scenario`, `Summary`, `Test Type`, `Priority`, `PreConditions`, `Test Step`, `Test Data`, `Expected Result`, `Actual Result`, `Test Case Status`, `Automation Status`, `Notes`. Notion page/database links are stored in `planning_state`, handoff contracts, and other active platform sync outputs.

## Outputs

The source of truth is `planning_state.json`. Human-readable plans, test cases, coverage maps, spreadsheet outputs, and downstream handoff contracts are rendered from that state.

Expected outputs include:
- test plan
- test cases using the `Template Test Case.xlsx` field model
- requirement coverage map
- `qa-executor`, `qa-automation`, and `qa-reporter` handoff contracts
- Plane comment, attachment, wiki/page, page link, sync record, and optional status transition output when Plane tools are available
- Notion test plan page, Notion test case database, Notion views, and captured Notion links when Notion tools are available
- review history and changelog entries

## Usage Prompts

### Basic QA Planning

```text
Gunakan skill qa-planner.

Buat QA planning package dari requirement berikut:
<isi requirement / PRD / user story / acceptance criteria>

Output yang dibutuhkan:
- planning_state.json
- test plan
- test cases
- coverage map
- risk analysis
- open questions jika ada informasi yang kurang jelas

Jangan gunakan asumsi. Jika ada informasi blocking yang kurang atau konflik, tanyakan dulu.
```

### Test Cases Only

```text
Gunakan skill qa-planner.

Buat test cases dari requirement berikut:
<isi requirement>

Gunakan field:
- TC ID
- Scenario
- Summary
- Test Type
- Priority
- PreConditions
- Test Step
- Test Data
- Expected Result
- Actual Result
- Test Case Status
- Automation Status
- Notes

Actual Result harus kosong.
Test Case Status default: Untested.
Jangan gunakan asumsi untuk expected result.
```

### Test Plan Only

```text
Gunakan skill qa-planner.

Buat test plan dari requirement berikut:
<isi requirement>

Ikuti struktur:
- Metadata
- Objective
- Source Inputs
- Scope
- Assumptions and Open Questions
- Risk Analysis
- Test Strategy
- Environment
- Entry Criteria
- Exit Criteria
- Dependencies
- Coverage Summary
- Handoff Summary

Jika ada informasi blocking yang kurang, tanyakan dulu.
```

### Plane Work Item

```text
Gunakan skill qa-planner.

Ambil requirement dari Plane work item:
<ENG-42 atau URL Plane>

Baca juga:
- description
- comments
- activities
- relations
- attachments jika readable
- module/cycle context jika tersedia

Buat:
- planning_state.json
- test plan
- test cases
- coverage map
- handoff contract

Tulis hasil planning kembali ke Plane work item sumber.
Ikuti Plane QA state workflow:
- default mulai dari `Todo Test`
- pindahkan ke `Analyze Test` saat mulai analisis
- pindahkan ke `Update Test` jika artifact perlu dibuat/diperbarui
- pindahkan ke `Need Review Test` setelah sync selesai
- pindahkan ke `Ready to Test` setelah user memberi OK
```

### Plane Batch Module Or Cycle

```text
Gunakan skill qa-planner.

Buat QA planning untuk Plane module/cycle berikut:
<module id / cycle id / URL Plane>

Baca semua work items di scope tersebut.
Perhatikan relation blocking/blocked_by.
Jika ada item yang blocked, masukkan dependency ke entry criteria.

Output:
- planning_state.json
- test plan
- test cases per work item/module
- coverage map
- risk analysis
- handoff contract
- summary comment ke Plane
```

### Notion Test Plan And Test Case Database

```text
Gunakan skill qa-planner.

Buat QA planning dari requirement berikut:
<isi requirement>

Output ke Notion:
- test plan sebagai Notion page
- test cases sebagai Notion database

Parent Notion page:
<URL atau page id Notion>

Kolom database test cases harus urut:
1. TC ID
2. Scenario
3. Summary
4. Test Type
5. Priority
6. PreConditions
7. Test Step
8. Test Data
9. Expected Result
10. Actual Result
11. Test Case Status
12. Automation Status
13. Notes

Simpan/copy link Notion page dan database di output akhir.
```

### Plane And Notion

```text
Gunakan skill qa-planner.

Ambil requirement dari Plane work item:
<ENG-42 atau URL Plane>

Buat QA planning package dan publish ke Notion:
- test plan sebagai Notion page
- test cases sebagai Notion database

Parent Notion page:
<URL atau page id Notion>

Setelah Notion page/database dibuat:
- copy link Notion page dan database
- attach/update link tersebut ke Plane work item sumber
- tambahkan summary comment di Plane

Ikuti Plane QA state workflow. Jika work item bukan di `Todo Test`, berhenti setelah membaca state lalu minta konfirmasi saya sebelum lanjut.
Jangan gunakan asumsi jika requirement kurang jelas.
```

### Update Existing Notion QA Plan

```text
Gunakan skill qa-planner.

Update QA planning berdasarkan feedback berikut:
<OK / NOK / partial feedback>

Existing Notion test plan page:
<URL Notion page>

Existing Notion test case database:
<URL Notion database>

Source requirement:
<URL / teks requirement / Plane item>

Update page dan database yang sama.
Jangan buat duplikat.
Catat perubahan di changelog.
```

### Review Feedback

```text
Gunakan skill qa-planner.

Review feedback:
<NOK: bagian test cases kurang negative scenario>
atau
<scope OK, test cases perlu revisi>
atau
<OK>

Existing planning package:
<planning_state.json / Notion link / Plane link / teks existing output>

Jika feedback partial, revisi hanya bagian yang terdampak.
Preserve bagian yang sudah approved.
```

### Handoff To QA Executor

```text
Gunakan skill qa-planner.

Dari QA planning package berikut:
<planning_state.json / test cases / Notion database / Plane item>

Buat handoff contract untuk qa-executor.

Isi contract:
- selected test cases
- run policy
- environment
- test data
- constraints
- gaps
- coverage refs
- source links

Jangan ubah status execution karena belum ada hasil test.
```

### Handoff To QA Automation

```text
Gunakan skill qa-planner.

Dari test cases berikut:
<test cases / planning_state.json / Notion database>

Buat handoff contract untuk qa-automation.

Klasifikasikan:
- To Be Automate
- Already Automate
- Manual Only

Jelaskan alasan automation_status untuk borderline cases.
Jangan tulis automation script final.
```

### Handoff To QA Reporter

```text
Gunakan skill qa-planner.

Dari QA planning package berikut:
<planning_state.json / Notion page / Plane item>

Buat handoff contract untuk qa-reporter.

Sertakan:
- planning summary
- coverage refs
- risk refs
- scope
- exclusions
- open questions
- approval status

Jangan buat final execution report.
```

### Full Prompt

```text
Gunakan skill qa-planner.

Source requirement:
<teks / PRD / Plane item / Notion page>

Buat QA planning package:
- planning_state.json
- test plan
- test cases
- coverage map
- risk analysis
- open questions
- handoff contracts untuk qa-executor, qa-automation, qa-reporter

Publish output:
- Notion test plan page
- Notion test case database
- copy link Notion page/database

Parent Notion page:
<URL>

Jika source dari Plane:
- baca comments, activities, relations, attachments
- attach/update link Notion ke Plane
- ikuti Plane QA state workflow
- jika work item Plane bukan di `Todo Test`, berhenti setelah membaca state lalu minta konfirmasi saya sebelum lanjut

Rules:
- jangan gunakan asumsi untuk requirement yang kurang jelas
- jika ada konflik antar source, tanyakan dulu
- Actual Result kosong
- Test Case Status default Untested
- ikuti urutan kolom database Notion yang sudah ditentukan
```

## Notion MCP Output

When Notion output is requested, `qa-planner` creates the test plan as a Notion page and test cases as a Notion database. The test plan page follows `templates/test-plan.md` with clean headings, compact tables, and bullet lists. Each test case becomes a database row/page with ordered display properties: `TC ID`, `Scenario`, `Summary`, `Test Type`, `Priority`, `PreConditions`, `Test Step`, `Test Data`, `Expected Result`, `Actual Result`, `Test Case Status`, `Automation Status`, and `Notes`. The default view, `All Cases`, and every qa-planner-created view must use that same visual column order; if a view cannot be updated or verified, `qa-planner` records the Notion view order gap.

The planner captures `test_plan_page_url` and `test_case_database_url` in `planning_state.notion_context` and mirrors them into `artifact_outputs` and handoff contracts. If database creation or schema update is unavailable, the planner creates a fallback Notion page containing a table with the same test case columns and records a Notion database/schema gap.

## Plane Hybrid Sync

Default Plane write mode is `full_sync`: add or update a concise managed comment on the source work item/card, attach generated artifacts to that source item, create a project/workspace wiki page when useful, link external or Plane pages back to the work item/card, and apply the Plane QA state workflow.

Plane MCP write behavior is deterministic: `create_work_item_comment` / `update_work_item_comment` handle summary comments, `retrieve_project_page` / `retrieve_workspace_page` read existing Plane pages when available, `create_project_page` / `create_workspace_page` publish full HTML plans, optional `update_project_page` / `update_workspace_page` update managed page sections when the MCP client supports it, `create_work_item_link` attaches durable plan links, and `update_work_item` performs dynamic status or label updates only after resolving valid project states. If page update tooling is unavailable, qa-planner creates a versioned replacement page, links it, and records `plane_page_update_gap`. Handoff tracking can create QA execution tracker items with `create_work_item` and place them into a QA cycle with `add_work_items_to_cycle` only when requested or required by an approved handoff contract.

Plane QA state transitions are:
- `Todo Test` -> `Analyze Test` when qa-planner starts analysis/planning.
- `Analyze Test` -> `Ready to Test` when requested artifacts already exist and are complete.
- `Analyze Test` -> `Update Test` when requested artifacts are missing, incomplete, outdated, or not traceable.
- `Update Test` -> `Need Review Test` after qa-planner creates/updates artifacts and syncs output.
- `Need Review Test` -> `Update Test` when user gives feedback, revision request, or `NOK`.
- `Need Review Test` -> `Ready to Test` when user approves or gives `OK`.

If the target state is not found, planning and Plane output still complete when possible, state movement is skipped, and the sync record notes `status_transition_skipped`. If Plane tools are unavailable, the planner falls back to JSON and Markdown outputs and records the sync gap.

## Review Loop

`OK` review sets `planning_status` to `approved`. `NOK` review records structured feedback, updates impacted planning state sections, and re-renders impacted artifacts until review passes.

## Downstream Handoff

- `qa-executor`: receives selected approved test cases, run policy, environment, test data, and coverage refs.
- `qa-automation`: receives automation candidates, existing automation updates, manual-only cases, and automation gaps.
- `qa-reporter`: receives planning summary, coverage refs, risk refs, planned scope, exclusions, and approval metadata.

`qa-planner` does not execute tests, create defect tickets, write final automation scripts, or produce final execution reports.
