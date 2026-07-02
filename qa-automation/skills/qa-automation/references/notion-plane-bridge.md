# Notion + Plane Bridge Reference

Load this file only when the same automation request uses both Notion and Plane, or when a Plane work item/comment/link contains a Notion URL.

## Activation

Activate the bridge when:
- Plane work item contains Notion links
- user provides Plane and Notion references in one request
- planner/executor handoff includes both refs
- user asks to update Notion and Plane together

## Source Of Truth

`automation_change_set.json` remains source of truth.

Plane is the workflow surface. Notion is the test case/status source and optional detailed automation tracking surface.

## Discovery And Classification

Scan Plane title, description, comments, links, and page/wiki content for Notion URLs.

Classify Notion refs:
- `test_case_source`: contains TC ID and Automation Status or equivalent test case fields
- `automation_page`: managed QA automation output page
- `result_database`: stores automation result rows
- `unknown`: role is unclear

If exactly one high-confidence Notion test case source is discovered, use it automatically. If multiple candidates exist or confidence is low, ask the user to choose.

## Cross-Linking

When policy allows:
- Plane comment/description should include Notion source/status/page link.
- Notion comments/pages/rows should include Plane work item link.

If detailed automation output exists in Notion, keep Plane output summary-only. Do not duplicate long validation tables, patch details, or review packages in Plane.

## Idempotency

Use bridge key:

```text
qa-automation:<change_set_id>:plane:<plane_ref>:notion:<notion_ref>
```

Bridge statuses:
- `not_started`
- `synced`
- `partial`
- `failed`
- `skipped`

Partial bridge sync does not invalidate local automation output. Record errors and continue with whichever side synced successfully.

## Safety

Apply both Notion and Plane redaction rules. Delete/archive still requires explicit approval on the target system. Do not create follow-up work items automatically for gaps.
