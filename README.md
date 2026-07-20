# GitHub Actions

Centralized GitHub Actions for tankdonut repositories.

## Actions

| Action | Description |
| ------ | ----------- |
| [install-asdf-dependencies](actions/install-asdf-dependencies/README.md) | Install tools from `.tool-versions` via asdf with caching |
| [pre-commit](actions/pre-commit/README.md) | Execute pre-commit hooks via asdf |
| [setup-python-uv](actions/setup-python-uv/README.md) | Setup Python and uv toolchain |
| [ghcr-login](actions/ghcr-login/README.md) | Login to GHCR (podman/docker/crane) |
| [git-bot-config](actions/git-bot-config/README.md) | Configure git identity for bot commits |
| [renovate-auto-approve](actions/renovate-auto-approve/README.md) | Approve non-draft PRs opened by Renovate bots |

## Reusable Workflows

| Workflow | Description |
| -------- | ----------- |
| [prune-ghcr](.github/workflows/prune-ghcr.yaml) | Prune stale GHCR container images |

## Versioning and releases

Actions are referenced via the `@v1` major tag. The `v1` tag moves forward with each compatible change, so consumers always get the latest patch without pinning a specific commit.

Third-party actions used inside composite actions (like `actions/checkout`, `actions/setup-python`) are SHA-pinned for supply-chain security. Renovate tracks these pins and opens PRs when new versions drop.

### Cutting a release

After merging PRs to `main`, move the `v1` tag and refresh the GitHub Release.

```bash
# 1. Sync main and review what's changing since the last v1 position
git checkout main && git pull --ff-only
git log --oneline $(git rev-parse v1)..main

# 2. Move the v1 tag (force-with-lease fails safely if someone else moved it)
OLD_V1=$(git rev-parse v1)
git tag -f -a v1 main -m "v1: <short description>"
git push --force-with-lease=v1:$OLD_V1 origin v1

# 3. Refresh the GitHub Release notes
gh release edit v1 --notes "<curated notes — see existing releases for format>"

# 4. Verify local and remote agree
git rev-parse v1                    # local
git ls-remote --tags origin v1      # remote — must match
```

Release notes are curated, not auto-generated. GitHub's `--generate-notes` reaches back to the previous *release* (or repo inception if none), which produces far too much noise on a moving tag. End each release body with a `Full Changelog` compare link scoped to the previous v1 SHA:

```
**Full Changelog**: https://github.com/tankdonut/github-actions/compare/<previous-v1-sha>...v1
```

## Conventions

### Composite action cross-references

When one composite action in this repo references another (e.g. `pre-commit` calls `install-asdf-dependencies`), use the full repo reference so external consumers can resolve it:

```yaml
uses: tankdonut/github-actions/actions/install-asdf-dependencies@v1
```

**Not** a local `./actions/install-asdf-dependencies` path — local paths resolve relative to the *consumer's* workspace, not this repo, and fail with `Can't find 'action.yaml' under /home/runner/work/<consumer>/<consumer>/actions/...`.

Local `./actions/<name>` references are still correct inside `.github/workflows/*.yaml` files (test workflows, production workflows) — those run inside this repo where the paths resolve against the repo's own checkout.

### Merge strategy

Prefer **merge commits** or **rebase merges** over squash merges for PRs that contain multiple atomic commits. GitHub's squash dialog can silently drop commits if the message or selection is edited — this happened in #23 (two of four commits went missing) and required two follow-up hotfix PRs (#27, #28) to repair consumer-visible breakage.

If squash-merging a multi-commit PR, verify the resulting main commit contains every file from every branch commit before moving `v1`.

## Contributing

Add new actions under `actions/`. Each action needs three files:

1. `action.yaml` - composite action definition
2. `README.md` - usage example with `@v1` reference and inputs table
3. `.github/workflows/test-{action-name}.yaml` - test workflow
