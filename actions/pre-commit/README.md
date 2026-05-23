# pre-commit GitHub Action

## Usage

```yaml
name: pre-commit

on:
  push:
    branches:
      - "main"
  pull_request:

jobs:
  pre-commit:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v6
        with:
          fetch-depth: 0

      - name: Execute pre-commit
        uses: tankdonut/github-actions/actions/pre-commit@v1
```

## Inputs

| Input | Description | Default |
|-------|-------------|---------|
| `extra_args` | Extra arguments to pass to pre-commit | `""` |
| `skip_hooks` | Space-separated list of hooks to skip | `""` |
| `asdf_dir` | Path to asdf installation directory for caching | `"~/.asdf"` |
