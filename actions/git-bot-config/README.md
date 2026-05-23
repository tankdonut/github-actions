# Git Bot Config

Configures git identity for automated commits using the canonical GitHub Actions bot identity.

## Usage

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v6

      - name: Configure git bot identity
        uses: tankdonut/github-actions/actions/git-bot-config@v1
```

## Inputs

| Name  | Default | Description |
|-------|---------|-------------|
| `name` | `github-actions[bot]` | Git user name |
| `email` | `41898282+github-actions[bot]@users.noreply.github.com` | Git user email (canonical GitHub bot format) |

### Canonical email format

The default email uses the canonical `41898282+` prefix required for GitHub to attribute commits to the `github-actions[bot]` user.
