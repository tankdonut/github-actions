# Renovate Auto-Approve

Approves non-draft pull requests opened by Renovate bots using a PAT (Personal Access Token). The PAT is required because GitHub's default `GITHUB_TOKEN` cannot approve PRs that were opened by workflow runs (a self-approval guard).

The action wraps [`hmarr/auto-approve-action`](https://github.com/hmarr/auto-approve-action) and adds a token-presence check that fails fast with a clear error message if the PAT is missing.

## Usage

```yaml
name: Renovate Auto-Approve

on:
  pull_request:
    types: [opened, ready_for_review, synchronize]

# Prevent concurrent approvals on the same PR.
concurrency:
  group: renovate-auto-approve-${{ github.event.pull_request.number }}
  cancel-in-progress: false

permissions:
  pull-requests: write

jobs:
  approve:
    # Only approve non-draft PRs opened by Renovate. Drafts (e.g. major bumps)
    # remain pending human review until marked ready.
    if: |
      github.event.pull_request.draft == false &&
      (github.actor == 'renovate[bot]' || github.actor == 'renovate-config[bot]')
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v6

      - name: Approve Renovate pull request
        uses: tankdonut/github-actions/actions/renovate-auto-approve@v1
        with:
          github-token: ${{ secrets.AUTO_APPROVE_PAT }}
```

## Inputs

| Input | Required | Default | Description |
| ----- | -------- | ------- | ----------- |
| `github-token` | yes | — | PAT with `pull_request:write` scope. Pass via a secret like `secrets.AUTO_APPROVE_PAT`. |
| `review-message` | no | `Auto-approved by Renovate Auto-Approve workflow. Merge is gated by required status checks via GitHub Auto-Merge.` | Message attached to the approval review |

## What stays in the consumer workflow

A composite action cannot declare workflow-level configuration, so the following **must** remain in the calling workflow:

- `on: pull_request` triggers
- `concurrency:` group (prevents racing approvals on the same PR)
- `permissions: pull-requests: write`
- The `if:` job-level gate (draft status + `github.actor` matching Renovate bots)

## Notes

- **Why a PAT, not `GITHUB_TOKEN`:** GitHub blocks the default `GITHUB_TOKEN` from approving PRs that were created by a workflow run (Renovate PRs qualify), so a PAT is required.
- **Auto-merge:** This action only approves. To enable auto-merge, configure GitHub's Auto-Merge feature on the repository (or via `gh` CLI) separately.
- **Draft handling:** Major Renovate bumps typically open as drafts. The `if:` gate in the consumer workflow ensures drafts aren't auto-approved until they're marked ready for review.
