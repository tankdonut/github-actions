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
