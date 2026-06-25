# Plane Hybrid Reference

Load this file only when input contains Plane.so signals such as a Plane work item/card, readable id like `ENG-42`, Plane URL/UUID/payload, Plane comment, attachment, page/wiki, or Plane Agent @mention.

## Plane Intake

Accept Plane input from:
- work item/card title and description
- comments and review replies
- readable attachment content
- project pages or wiki pages
- parent, child, linked work items
- module and cycle context
- direct MCP/API payloads

Capture available refs in `planning_state.source_inputs[].plane_refs` using `schemas/plane-source.schema.json` when possible: workspace, project, work item UUID, readable identifier, source URL, source status, comments, attachments, pages, modules, cycles, and links.

For Plane QA planning, source status controls whether qa-planner may proceed. Follow the Plane QA state workflow below before analysis, generation, or status movement.

## Plane QA State Workflow

Use this workflow whenever input contains Plane context such as `Plane`, `Plane.so`, Plane work item/card, readable issue id like `DKI-179`, Plane URL/UUID, or a request to read, analyze, create, or update QA artifacts from a Plane work item.

Known QA states:
- `Backlog Test Human`
- `Todo Test`
- `Analyze Test`
- `Update Test`
- `Need Review Test`
- `Ready to Test`

Resolve state names and ids with `list_states` before any transition. If a state is missing, record `plane_state_gap` and ask the user instead of guessing a replacement state.

State ownership and entry rules:
- `Backlog Test Human`: human-owned queue after developer completion in dev environment. Read only when the user explicitly requests this exact item or state. Do not auto-pull work from this state.
- `Todo Test`: default eligible state for qa-planner intake. If the user asks qa-planner to start planning or analysis from this state, move the work item to `Analyze Test`.
- `Analyze Test`: qa-planner analysis state. Read and decide routing only; do not create or heavily update test cases/test plans here.
- `Update Test`: qa-planner generation/update state. Use for creating or updating requested QA artifacts.
- `Need Review Test`: human review state after qa-planner finishes creating/updating artifacts.
- `Ready to Test`: artifacts are ready for qa-executor. Do not change test cases/test plans in this state unless the user explicitly requests revision.

Default state gate:
- By default, process only work items in `Todo Test`.
- If the work item is in any other state, stop after reading the current state and ask the user for confirmation before analysis, generation, or workflow transition.
- Mentioning a specific item id such as `DKI-213`, saying "planning langsung", or explicitly naming a non-`Todo Test` state is not confirmation.
- Proceed only after the user gives clear approval such as `OK`, `approve`, `lanjut`, or equivalent confirmation after being told the current non-`Todo Test` state.
- Record the confirmation in `planning_state.review_history` or `custom_fields.plane_state_confirmation`.

State machine:
- `Todo Test` -> `Analyze Test` when user asks qa-planner to start analysis/planning.
- `Analyze Test` -> `Ready to Test` when all requested artifacts already exist, are complete, current, and traceable to requirements.
- `Analyze Test` -> `Update Test` when any requested artifact is missing, incomplete, outdated, not traceable, or has important gaps.
- `Update Test` -> `Need Review Test` after qa-planner finishes creating/updating artifacts and syncing output to Plane.
- `Need Review Test` -> `Update Test` when user gives feedback, revision request, or `NOK`.
- `Need Review Test` -> `Ready to Test` when user approves or gives `OK`.

`Analyze Test` behavior:
- Read and analyze issue title, description, parent, labels, priority, attachments, comments, related context, existing test cases, existing test plan, and other QA artifacts relevant to the user request.
- Match analysis scope to the user request. If the user asks for test cases, check existing test cases. If the user asks for test plan, check existing test plan. If both are requested, check both.
- If all requested artifacts already exist and satisfy the work item requirements, move to `Ready to Test`.
- If artifacts are missing, incomplete, outdated, not traceable, or contain important gaps, move to `Update Test`.
- Do not perform large artifact creation or update while still in `Analyze Test`.

`Update Test` behavior:
- Create or update only the artifacts requested by the user: test cases, test plan, coverage map, or other QA artifacts.
- Sync results to Plane using the active Plane output policy.
- Record artifacts created, artifacts updated, change summary, gaps/open questions, and coverage summary.
- After successful sync, move the work item to `Need Review Test`.

`Need Review Test` behavior:
- If user feedback is ambiguous, ask for clarification or record a blocking open question before moving state.
- If user gives revision feedback or `NOK`, move to `Update Test`, revise impacted artifacts, sync, then return to `Need Review Test`.
- If user gives approval or `OK`, move to `Ready to Test`.

`Ready to Test` behavior:
- Enter only when requested artifacts exist, are sufficiently complete against requirements, no blocking gap remains, and user approved or the user flow allows auto-ready after valid analysis.
- After entering this state, qa-planner must not change test cases or test plan again unless the user explicitly requests revision.

## Plane MCP Tool Map

Use Plane MCP tools when available. If a required tool is missing or fails, record `plane_mcp_gap` with tool name, source ref, and impact; ask the user only when the missing data blocks accurate planning.

Intake and scope tools:
- `retrieve_work_item_by_identifier`: resolve readable ids such as `ENG-42` into full work item payload before asking the user to paste details.
- `retrieve_work_item`: load a work item by UUID when project and work item ids are already known.
- `list_work_item_comments`: read requirement discussion, review replies, decisions, and clarifications before deriving test cases.
- `list_work_item_activities`: read change history; record changed acceptance criteria, priority, state, assignee, labels, cycle, module, or due date as source context.
- `list_cycle_work_items`: expand sprint/cycle scope when the requirement points to a cycle or sprint.
- `list_module_work_items`: expand feature/module scope when the requirement points to a module.
- `list_work_item_links`: detect existing PRD, design, QA plan, Google Docs, Confluence, Figma, API docs, or release notes before creating new links.

Planning enrichment tools:
- `list_work_item_relations`: capture `blocking`, `blocked_by`, `duplicate_of`, `duplicate`, and `relates_to` dependencies.
- `list_work_item_properties`: read project custom fields and preserve relevant values in `planning_state.plane_context.work_item_properties`.
- `list_work_item_types`: read project work item types and map them to planning scope, not to arbitrary `test_type` values unless explicitly defined by project convention.
- `get_project_members`: resolve assignee names/ids and available project members; do not invent QA owner or reviewer.
- `list_states`: resolve valid state ids before any status movement; never hard-code `In Review` as the only target.

Output and handoff tools:
- `create_work_item_comment` / `update_work_item_comment`: write or update the managed QA planning summary comment.
- `create_project_page` / `create_workspace_page`: publish full QA plan as HTML wiki/page output when requested, long, reusable, or useful for module/cycle scope.
- `update_work_item`: move status, update labels, or update fields only when explicitly requested and after successful sync.
- `create_work_item_link`: attach external test plan links such as Google Docs, Confluence, or other durable docs to the source work item.
- `create_work_item`: create QA execution tracker work items or sub-items for test case batches only when user requests Plane handoff tracking.
- `add_work_items_to_cycle`: place created QA execution tracker items into the configured QA cycle/sprint.

Preferred intake sequence for a readable id:
1. Split `ENG-42` into `project_identifier = ENG` and `work_item_identifier = 42`.
2. Call `retrieve_work_item_by_identifier`.
3. From returned UUIDs, call `list_work_item_comments`, `list_work_item_activities`, `list_work_item_relations`, `list_work_item_links`, `list_work_item_properties`, `list_work_item_types`, `get_project_members`, and `list_states` as needed.
4. If the item belongs to a cycle or module and user requested sprint/module scope, call `list_cycle_work_items` or `list_module_work_items` and plan the batch.
5. Normalize payload, comments, activities, relations, project members, states, properties, and types into `planning_state` before test case generation.

Do not call write tools until planning artifacts are ready, redaction checks pass, and the selected Plane write mode allows writes.

## Planning State Enrichment

Record Plane-derived context in `planning_state.plane_context` when schema support exists, otherwise use `custom_fields.plane_context` with the same shape.

Minimum shape:

```json
{
  "project_members": [],
  "states": [],
  "work_item_properties": [],
  "work_item_types": [],
  "relations": [],
  "batch_scope": []
}
```

Relation handling:
- `blocked_by`: add the blocking work item as a `test_plan.entry_criteria` dependency before execution of affected test cases.
- `blocking`: record risk and coverage impact for downstream items that depend on this work item.
- `relates_to`: inspect related items for shared rules or regression impact; add cases only when source evidence supports it.
- `duplicate` / `duplicate_of`: avoid duplicate test cases; map coverage to the canonical item when clear, otherwise ask.

Project metadata handling:
- Use `list_states` output to validate any requested status target dynamically.
- Use `get_project_members` output to resolve owner/reviewer/assignee references.
- Use `list_work_item_properties` and `list_work_item_types` to preserve project-specific fields and type context; ask before changing the controlled QA `test_type` enum.

## Attachment Intake

Read or download Plane attachment content when tools and file type allow it. Treat readable text as requirement input.

If attachment content cannot be read:
- record `attachment_read_gap` with attachment id/name
- ask the user for attachment content when it is needed for accurate planning
- do not infer missing attachment details

## Status-Move Clarification

When Plane QA state workflow is active, use its defined transitions instead of asking a generic status-move question.

For Plane work that is not using the QA state workflow and the user does not specify status movement, ask this before running the planning workflow:

```text
Setelah QA planning selesai dan output berhasil diattach/update ke Plane, apakah work item perlu dipindahkan status? Jika ya, ke status apa?
```

Do not ask only when the user already provided a target status, explicitly said not to move status, selected `read_only`, or no Plane write tool is available and the run is fallback-only.

If the user says no, continue without status movement and set transition status to `not_requested`. If the user says yes but target status is unclear, ask one concise follow-up before running.

## Plane Output Policy

Default write mode for Plane input is `full_sync`.

Modes:
- `read_only`: generate artifacts without writing to Plane
- `comment_only`: write concise summary comment to the source work item/card
- `attach_artifacts`: comment plus artifacts attached to the source work item/card
- `wiki_update`: create/update Plane wiki/page and link it when supported
- `full_sync`: comment, attachments, wiki/page when useful, page link, then optional status movement after successful sync

When Plane write tools are available, output must be added or updated on the source work item/card. Returning output only in chat is not enough.

Managed comment idempotency:
- Before creating a new comment, read existing comments with `list_work_item_comments` and look for the managed marker `qa-planner-managed` plus package id.
- If found, update that comment with `update_work_item_comment` when available; otherwise append a concise new version comment.
- If not found, create one comment with `create_work_item_comment`.

Wiki/page publishing:
- Use `create_project_page` for project-scoped plans and `create_workspace_page` for cross-project plans.
- Page body must be HTML when the tool expects `description_html`.
- Preserve manual content by writing only inside managed sections when updating is available; if update is not available, create a new versioned page and link it.

External plan links:
- If a durable external plan URL exists or is created outside Plane, attach it with `create_work_item_link`.
- Check `list_work_item_links` first to avoid duplicate links for the same package id or URL.

## Comment, Attachment, and Wiki Behavior

Use `templates/plane-comment.md` for concise comments. Include package id/version, source item, planning status, test counts, open gaps/questions, artifact refs, wiki/page link, status transition result, and reviewer next action.

Attach JSON and Markdown at minimum. Attach Excel/PDF/Word when generated and upload tools exist. Upload changed artifacts only when possible.

Use `templates/plane-page.md` when the plan is long, has many test cases, is reusable across work items/modules/cycles, is requested by the user, or should become QA knowledge. Preserve manual page content and update only managed sections.

## Status Movement

Move status only when the Plane QA state workflow requires it, the user explicitly requests it, or the configured Plane output policy allows it. For post-update transitions, move only after configured Plane sync succeeds. Sync success means required comment, attachment, wiki/page, and link actions completed.

Resolve target state with `list_states`. Do not assume `In Review`; accept any user-provided valid target state name or id in the project.

Use `update_work_item` for state movement or label/field updates. Skip state movement when target state is not found, sync fails, transition tooling is unavailable, or the user explicitly chose no move for a non-workflow run. If target state is not found, complete planning and output sync when possible, record `status_transition_skipped`, and include a short reason in the Plane comment.

Status movement is not QA approval. `planning_status = approved` is controlled by review.

## Idempotency

Use `schemas/plane-sync-record.schema.json` to avoid duplicate Plane output.

Track package id, source agent, external id, source refs, managed comment refs, attachment refs, wiki/page refs, page links, artifact checksums, artifact versions, sync status, transition status, gaps, and errors.

On rerun, update existing managed comment/page when possible, upload new attachments only when content or version changed, and append changelog instead of duplicating the full plan.

## Plane Handoff Tracking

Use Plane handoff tracking only when requested by the user or when the approved handoff contract explicitly requires Plane tracker items.

Execution tracker behavior:
- Group approved test cases into batches by module, risk, test type, environment, or cycle.
- Use `create_work_item` to create one QA execution tracker item per batch, not one item per small test case unless user asks.
- Use parent/sub-item fields when available to keep tracker items under the source work item or QA epic.
- Add created tracker item ids to `handoff_contracts.qa_executor` under Plane context or `custom_fields.plane_execution_items`.
- Use `add_work_items_to_cycle` only when a QA cycle/sprint id is known or user selected one.
- Do not create execution tracker items before planning is approved unless the user explicitly asks for draft handoff tracking.

## Safety

Before writing to Plane, check for secrets, tokens, passwords, cookies, API keys, credentials, private keys, PII, and sensitive test account details. Redact or ask for approval before writing sensitive data to shared Plane surfaces.

If Plane write fails, return local/Markdown/JSON output, record `plane_sync_gap`, and do not move status.
