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
      - name: Execute pre-commit
        uses: tankdonut/github-actions/actions/pre-commit@v1
```
