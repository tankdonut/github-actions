# Setup Python + uv

Sets up a Python and [uv](https://docs.astral.sh/uv/) toolchain for CI workflows. Uses `actions/setup-python` and `astral-sh/setup-uv` under the hood.

## Inputs

| Input | Default | Description |
| --- | --- | --- |
| `python-version` | `3.13` | Python version to install |
| `uv-version` | `latest` | uv version to install |

## Usage

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v6

      - name: Setup Python + uv
        uses: tankdonut/github-actions/actions/setup-python-uv@v1

      - name: Install dependencies
        run: uv sync
        shell: bash
```

**Note:** `uv sync` (or any project-level install step) remains a caller responsibility since different repos have different dependency needs.
