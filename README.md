# Setup cosign Action

This GitHub Action installs [sigstore/cosign](https://github.com/sigstore/cosign) to the GitHub Actions runner tool cache and adds it to the PATH. It **always** verifies the downloaded binary before installation.

## Versioning

This action uses **immutable releases** with full semantic versioning (`vMAJOR.MINOR.PATCH`). Floating major tags (for example, `v1`) are **not** provided.

When referencing this action, either use the full version string or pin to a commit SHA:

```yaml
# Full version string
uses: rsclarke/setup-cosign@<VERSION>

# Pinned to SHA (recommended)
uses: rsclarke/setup-cosign@<COMMIT_SHA> # <VERSION>
```

## Platform Support

This action supports **Linux** and **macOS** runners only. Windows is not supported.

## Verification

The action bootstraps cosign installation from GitHub Releases using three artifacts:

- `cosign-<os>-<arch>`
- `cosign-<os>-<arch>-kms.sigstore.json`
- `release-cosign.pub`

It verifies the downloaded `release-cosign.pub` against a pinned expected public key, extracts the release signature from the KMS Sigstore bundle with `jq`, and verifies the cosign binary with `openssl` before installing it.

If Sigstore rotates the cosign artifact signing key, the action will fail closed until the pinned key in this repository is updated.

## Caching

The action uses [`actions/cache`](https://github.com/actions/cache) to persist the tool cache across workflow runs. On a cache hit the download and verification steps are skipped entirely, using a `.complete` sentinel file to validate cache integrity.

When cache usage is enabled, the cache key includes the runner OS, architecture, and resolved cosign version:

```
setup-cosign-{runner.os}-{runner.arch}-{version}
```

When no explicit version is provided, the latest release is resolved first so it still participates in caching. Set `use-cache: "false"` to skip cache restore/save and force a fresh download.

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `version` | Version of sigstore/cosign to install (for example, `v3.0.6`). If not specified, the latest release will be resolved automatically. | No | Latest |
| `use-cache` | Whether to restore and save the GitHub Actions cache for the cosign tool cache. Set to `false` to force a fresh download. | No | `true` |

## Outputs

| Output | Description |
|--------|-------------|
| `cosign_path` | Path to the installed `cosign` binary. |

The action uses the caller workflow's `GITHUB_TOKEN` automatically via `github.token`, so no token input is required.

To force a fresh download and verification on every run:

```yaml
- name: Install cosign from a fresh download
  uses: rsclarke/setup-cosign@<VERSION>
  with:
    use-cache: "false"
```

## Usage

```yaml
jobs:
  build:
    runs-on: ubuntu-latest # or macos-latest
    steps:
      - uses: actions/checkout@v6

      - name: Install cosign
        uses: rsclarke/setup-cosign@<VERSION>

      - name: Verify cosign
        run: cosign version
```

You can also use the action output directly:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - id: setup-cosign
        uses: rsclarke/setup-cosign@<VERSION>

      - name: Show installed path
        run: echo "Cosign installed at ${{ steps.setup-cosign.outputs.cosign_path }}"
```
