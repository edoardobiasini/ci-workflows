# ci-workflows

Centralized, reusable GitHub Actions workflows shared across Patterns Digital
projects. Consuming repos reference a workflow by tag so CI/deploy logic lives
here once and improvements propagate with a version bump — no copy-paste per
project.

## Workflows

### `.github/workflows/deploy.yml` — Cloudways build + rsync deploy

Builds in CI (Composer + npm) and rsyncs the theme and mu-plugin to a Cloudways
application, then runs a post-deploy sanity check, cache purge, and health check.

**Usage** (in a consuming repo, e.g. a Sage theme):

```yaml
name: Deploy to Cloudways
on:
  workflow_dispatch:
    inputs:
      environment:
        description: Target environment
        type: choice
        options: [staging, production]
        required: true

concurrency:
  group: deploy-${{ inputs.environment }}
  cancel-in-progress: false

jobs:
  deploy:
    uses: edoardobiasini/ci-workflows/.github/workflows/deploy.yml@v1
    with:
      environment: ${{ inputs.environment }}
    secrets: inherit
```

The caller owns the trigger and `concurrency`; the reusable workflow owns the
build/deploy steps. `secrets: inherit` + the `environment:` set inside the
reusable job is how the caller's environment-scoped `CW_*` variables and the
`CLOUDWAYS_ACCESS_TOKEN` / `DEPLOY_SSH_KEY` secrets resolve. See the header of
`deploy.yml` for the full list of required environment variables and secrets.

The reusable workflow's `actions/checkout` checks out the **caller's** repo, so
the caller must provide `bin/remote-post-deploy.sh`.

## Access

This repo is private. For a private caller to use these workflows, this repo's
**Settings → Actions → General → Access** must allow "Accessible from
repositories owned by the user" (set once).

## Versioning

Consumers pin to a moving major tag (`@v1`). Breaking changes to a workflow's
inputs/secrets contract get a new major tag (`v2`).
