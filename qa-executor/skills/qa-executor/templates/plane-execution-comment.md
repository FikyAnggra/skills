<!-- qa-executor:execution:<execution_id>:<plane_work_item_id> -->

# QA Execution Summary

- Execution ID: `<execution_id>`
- Package ID: `<package_id>`
- Run mode: `<run_policy.mode>`
- Environment: `<environment.name>`
- Status: `<plane_sync.status_before>` -> `<plane_sync.status_after>`
- Review status: `<review_status>`
- Detailed report: `<notion_plane_bridge.notion_execution_page_ref or execution_report_ref>`

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

- Worklog: `<plane_sync.worklog_id>`
- Notion report link synced: `<plane_to_notion_link_sync_status>`
- Sync status: `<plane_sync.sync_status>`
- Sync errors: `<plane_sync.sync_errors>`

Details live in the Notion execution page when present. Do not duplicate full test tables, detailed evidence, reproduction steps, or issue candidate details in Plane.
