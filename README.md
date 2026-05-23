# GitHub Actions

Centralized GitHub Actions for tankdonut repositories.

## Actions

| Action | Description |
| ------ | ----------- |
| [pre-commit](actions/pre-commit/README.md) | Execute pre-commit hooks via asdf |
| [setup-python-uv](actions/setup-python-uv/README.md) | Setup Python and uv toolchain |
| [ghcr-login](actions/ghcr-login/README.md) | Login to GHCR (podman/docker/crane) |
| [git-bot-config](actions/git-bot-config/README.md) | Configure git identity for bot commits |

## Reusable Workflows

| Workflow | Description |
| -------- | ----------- |
| [prune-ghcr](.github/workflows/prune-ghcr.yaml) | Prune stale GHCR container images |

## Versioning

Actions are referenced via the `@v1` major tag. The `v1` tag moves forward with each compatible change, so consumers always get the latest patch without pinning a specific commit.

Third-party actions used inside composite actions (like `actions/checkout`, `actions/setup-python`) are SHA-pinned for supply-chain security. Renovate tracks these pins and opens PRs when new versions drop.

## Contributing

Add new actions under `actions/`. Each action needs three files:

1. `action.yaml` - composite action definition
2. `README.md` - usage example with `@v1` reference and inputs table
3. `.github/workflows/test-{action-name}.yaml` - test workflow
