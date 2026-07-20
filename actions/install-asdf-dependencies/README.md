# Install asdf Dependencies

Installs tools declared in [`.tool-versions`](https://asdf-vm.com/manage/configuration.html#tool-versions) via [asdf](https://asdf-vm.com/), with caching of the asdf install directory and optional installation of Python build dependencies (`libbz2-dev`, `libreadline-dev`, `liblzma-dev`) needed for asdf to compile the `bz2`, `readline`, and `lzma` extension modules.

When Python is declared in `.tool-versions`, the action also appends `$(asdf where python)/bin` to `GITHUB_PATH` so that pip-installed binaries (e.g. `pre-commit`) are discoverable — asdf only puts shims on PATH, not install `bin/` directories.

## Usage

```yaml
name: ci

on:
  push:
    branches:
      - "main"
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v6

      - name: Install asdf tools
        id: asdf
        uses: tankdonut/github-actions/actions/install-asdf-dependencies@v1

      - name: Run a tool from .tool-versions
        if: steps.asdf.outputs.cache-hit != 'true'
        run: echo "Fresh install — first run after .tool-versions change"
```

No-op safe: if `.tool-versions` is absent, every step is skipped and the action completes successfully with empty outputs.

## Inputs

| Input | Default | Description |
| ----- | ------- | ----------- |
| `asdf_dir` | `~/.asdf` | Path to asdf installation directory for caching |
| `tool_versions_file` | `.tool-versions` | Path to `.tool-versions` file |
| `install_python_build_deps` | `true` | Install `libbz2-dev`, `libreadline-dev`, `liblzma-dev` via apt when Python is declared in `.tool-versions` on Linux runners |

## Outputs

| Output | Description |
| ------ | ----------- |
| `cache-hit` | `'true'` when the asdf cache was restored (no install was performed); empty if `.tool-versions` is absent |
| `python-in-tool-versions` | `'true'` if `python` is declared in `.tool-versions`, `'false'` otherwise; empty if `.tool-versions` is absent |

## Notes

- The Python build-dependency step is gated to Linux runners (uses `apt-get`). On macOS, the step is skipped — asdf-built Python on macOS may still miss the bz2/readline/lzma modules; install the Homebrew equivalents (`readline`, `xz`, `bzip2`) upstream if needed.
- The build-dependency step only runs on asdf cache miss — when the cache hits, the Python build already happened previously with the libraries available (or will use the cached binary).
- On cache hit, `asdf-vm/actions/install` is skipped entirely. To keep asdf usable in that case, the action adds `$asdf_dir/bin` and `$asdf_dir/shims` to `GITHUB_PATH` and exports `ASDF_DIR` / `ASDF_DATA_DIR` itself (mirroring what `asdf-vm/actions/install` does on a fresh install). This runs on both cache-hit and cache-miss paths.
