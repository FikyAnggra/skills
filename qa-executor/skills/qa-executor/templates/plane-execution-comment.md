<!-- qa-executor:execution:<execution_id>:<plane_work_item_id> -->

# QA Execution Completion / Review Summary

- Execution ID: `<execution_id>`
- Package ID: `<package_id>`
- Run mode: `<run_policy.mode>`
- Environment: `<environment.name>`
- Status: `<plane_sync.status_before>` -> `<plane_sync.status_after>`
- Comment timing: `<plane_sync.comment_timing>`
- Terminal outcome: `<plane_sync.terminal_comment_outcome>`
- Terminal comment status: `<plane_sync.terminal_comment_status>`
- Progress comment created: `<plane_sync.progress_comment_created>`
- Review status: `<review_status>`
- Detailed report: `<notion_plane_bridge.notion_execution_page_ref or execution_report_ref>`

## Plane Workflow

- Current state before execution: `<plane_state_workflow.current_state_before_execution>`
- Start gate: `Ready to Test`
- Execution state: `In Testing`
- Review state: `Need Review Test Execute`
- Final workflow state: `<plane_state_workflow.final_review_state>`
- Approval override: `<plane_state_workflow.approval_override.approved>`
- Fresh approval required: `<plane_state_workflow.approval_override.fresh_approval_required>`
- Initial prompt override ignored: `<plane_state_workflow.approval_override.initial_prompt_override_ignored>`
- Approval source turn: `<plane_state_workflow.approval_override.approval_source_turn>`
- Approval note: `<plane_state_workflow.approval_override.approval_text>`

## Counts

| Selected | Passed | Failed | Blocked | Untested | Retest |
| ---: | ---: | ---: | ---: | ---: | ---: |
| `<summary.selected>` | `<summary.passed>` | `<summary.failed>` | `<summary.blocked>` | `<summary.untested>` | `<summary.retest>` |

## Attention

- Failed TC IDs: `<failed_tc_ids>`
- Blocked TC IDs: `<blocked_tc_ids>`
- Blocker count: `<summary.blocked>`
- Redaction status: `<redaction_status>`

## Sync

- Source update summary: `<source_update_summary>`
- Source update gaps: `<source_update_gaps>`
- Worklog: `<plane_sync.worklog_id>`
- Notion report link synced: `<plane_to_notion_link_sync_status>`
- Sync status: `<plane_sync.sync_status>`
- Sync errors: `<plane_sync.sync_errors>`

Details live in the Notion execution page when present. Do not duplicate full test tables, detailed evidence, reproduction steps, or issue candidate details in Plane.
