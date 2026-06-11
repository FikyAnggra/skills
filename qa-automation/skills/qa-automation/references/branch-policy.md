# Branch Policy

Follow repository policy first. Inspect contributing docs, package scripts, CI config, current branch names, and existing PR conventions.

## Fallback

Use fallback `develop -> master` when no repo policy is discoverable and branch context is needed.

## Supported Flows

- Feature branch to develop.
- Feature branch to main or master.
- Direct local patch without remote action.
- PR flow with script review and promotion review.
- Custom branch policy when user provides one.

## Remote Actions

Never push, deploy, merge, tag, or alter remote branches without explicit user request and approval. Record approval state in `branch_policy.remote_action_approved` and `promotion_status.remote_action_approved`.
