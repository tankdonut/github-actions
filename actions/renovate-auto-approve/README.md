# Renovate Auto-Approve

Approves non-draft pull requests opened by Renovate bots using a PAT (Personal Access Token). The PAT is required because GitHub's default `GITHUB_TOKEN` cannot approve PRs that were opened by workflow runs (a self-approval guard).

The action wraps [`hmarr/auto-approve-action`](https://github.com/hmarr/auto-approve-action) and adds a token-presence check that fails fast with a clear error message if the PAT is missing.

## Repository setup (one-time, required)

Before this action can approve PRs, the consuming repository needs four things configured. None of them can be declared in code — they all live in repo settings or secrets.

1. **Allow GitHub Actions to approve pull requests.**
   Settings → Actions → General → Workflow permissions → tick "Allow GitHub Actions to create and approve pull requests". Without this, the approval API call is rejected regardless of the token's scopes.

2. **Create the `AUTO_APPROVE_PAT` secret.**
   Settings → Secrets and variables → Actions → New repository secret. Name it `AUTO_APPROVE_PAT` and paste a fine-grained PAT (or classic PAT with `public_repo` / `repo` scope depending on visibility) that has the `pull_request:write` permission for this repository. The token must belong to a user or bot that is not blocked by branch protection rules as a reviewer.

3. **Enable auto-merge (so approval triggers the merge).**
   Settings → General → Pull Requests → tick "Allow auto-merge". This exposes the auto-merge UI on each PR and lets Renovate request auto-merge when it opens the PR. Without it, the approval stands alone and a human still has to click merge.

4. **Enable automatic branch deletion (optional but recommended).**
   Settings → General → Pull Requests → tick "Automatically delete head branches". Keeps the branch list clean after auto-merge lands.

All four can also be set programmatically via the GitHub API or `gh` CLI for bootstrapping multiple repositories.

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
