# GHCR Login

Login to GitHub Container Registry using podman, docker, or crane.

## Usage

### Podman (default)

```yaml
- name: Login to GHCR
  if: github.event_name != 'pull_request'
  uses: tankdonut/github-actions/actions/ghcr-login@v1
```

### Docker

```yaml
- name: Login to GHCR
  if: github.event_name != 'pull_request'
  uses: tankdonut/github-actions/actions/ghcr-login@v1
  with:
    runtime: docker
```

### Crane

```yaml
- name: Login to GHCR
  if: github.event_name != 'pull_request'
  uses: tankdonut/github-actions/actions/ghcr-login@v1
  with:
    runtime: crane
```

## Inputs

| Input     | Default              | Description                                        |
| --------- | -------------------- | -------------------------------------------------- |
| `runtime` | `podman`             | Container runtime: podman, docker, or crane        |
| `registry`| `ghcr.io`            | Container registry URL                             |
| `username`| `${{ github.actor }}`| Registry username                                  |
| `token`   | `${{ github.token }}`| Registry password/token                            |

## Notes

- Wrap usage in `if: github.event_name != 'pull_request'` to avoid login failures on PRs from forks where `GITHUB_TOKEN` permissions differ.
- Note that `pull_request_target` events also pass that guard (`event_name` is `pull_request_target`, not `pull_request`) and run with base-repo credentials. If your workflow uses that trigger, gate login with `if: github.event_name != 'pull_request' && github.event_name != 'pull_request_target'` — the login persists credentials to the runner's auth file (`~/.docker/config.json` or `$XDG_RUNTIME_DIR/containers/auth.json`), where later steps that execute fork-controlled code could read them.
