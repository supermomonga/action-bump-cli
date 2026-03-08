# action-bump-cli

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-action--bump--cli-blue?logo=github)](https://github.com/marketplace/actions/action-bump-cli)

A GitHub Action that bumps semantic versions in any file using the [bump](https://github.com/mattn/bump) CLI.

Since it uses regex patterns to locate version strings, it works with any file format.

> [!NOTE]
> This project is heavily inspired by [r7kamura/bump-request](https://github.com/r7kamura/bump-request). While bump-request creates pull requests to bump versions, this action focuses on the version bumping step itself, giving you full control over your workflow.

## Inputs

| Input | Required | Description |
|-------|----------|-------------|
| `file` | Yes | Path to the file containing the version string |
| `pattern` | Yes | Regex with a capture group to match the version |
| `release_type` | No* | One of `major`, `minor`, or `patch` |
| `version` | No* | Explicit version string to set (e.g. `2.0.0`) |

\* `release_type` and `version` are mutually exclusive. Specify one or the other.

## Outputs

| Output | Description |
|--------|-------------|
| `version` | The new version string after bumping |

## Quick Start

The following workflow lets you bump the version via **workflow_dispatch** and automatically creates a pull request with the change.

```yaml
# .github/workflows/bump-version.yml
name: Bump version

on:
  workflow_dispatch:
    inputs:
      release_type:
        description: "Release type (major, minor, patch). Ignored if version is specified."
        required: false
        type: choice
        options:
          - patch
          - minor
          - major
      version:
        description: "Explicit version to set (e.g. 1.2.3). Takes precedence over release_type."
        required: false
        type: string

permissions:
  contents: write
  pull-requests: write

jobs:
  bump:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: supermomonga/action-bump-cli@v1
        id: bump
        with:
          file: package.json
          pattern: '"version":\s*"(\d+\.\d+\.\d+)"'
          release_type: ${{ inputs.version == '' && inputs.release_type || '' }}
          version: ${{ inputs.version }}

      - uses: peter-evans/create-pull-request@v7
        with:
          branch: release/v${{ steps.bump.outputs.version }}
          commit-message: "Bump version to ${{ steps.bump.outputs.version }}"
          title: "Release v${{ steps.bump.outputs.version }}"
          body: |
            Bumps version to `${{ steps.bump.outputs.version }}`.
```

### Granting PR Creation Permission to GitHub Actions

This workflow requires GitHub Actions to have write access so it can create pull requests. You need to **enable read and write permissions** in your repository settings.

Go to **Settings > Actions > General > Workflow permissions** and select "Read and write permissions", or run the following `gh` command:

```bash
gh api repos/{owner}/{repo}/actions/permissions/workflow \
  --method PUT \
  --field default_workflow_permissions=write \
  --field can_approve_pull_request_reviews=false
```

> [!NOTE]
> Replace `{owner}/{repo}` with your actual repository (e.g. `supermomonga/my-app`).

## Pattern Examples

| File Type | Pattern |
|-----------|---------|
| package.json | `"version":\s*"(\d+\.\d+\.\d+)"` |
| Cargo.toml | `version\s*=\s*"(\d+\.\d+\.\d+)"` |
| .gemspec / version.rb | `VERSION\s*=\s*['"](\d+\.\d+\.\d+)['"]` |
| .csproj | `<Version>(\d+\.\d+\.\d+)</Version>` |
| pyproject.toml | `version\s*=\s*"(\d+\.\d+\.\d+)"` |
| VERSION (plain text) | `(\d+\.\d+\.\d+)` |

The pattern must contain exactly **one capture group** that matches the version string.

## Chaining: Bump Multiple Fields

You can use the output version in subsequent steps to update multiple version fields in the same file.

```yaml
# Bump <Version> and get the new version
- uses: supermomonga/action-bump-cli@v1
  id: bump
  with:
    file: MyApp.csproj
    pattern: '<Version>(\d+\.\d+\.\d+)</Version>'
    release_type: minor

# Set <FileVersion> to the same version
- uses: supermomonga/action-bump-cli@v1
  with:
    file: MyApp.csproj
    pattern: '<FileVersion>(\d+\.\d+\.\d+)</FileVersion>'
    version: ${{ steps.bump.outputs.version }}
```

## Example Workflows

See the [`examples/`](examples/) directory for complete workflow examples.

- [RubyGem](examples/rubygem.yml) — Bump the `VERSION` constant in `version.rb`
- [npm](examples/npm.yml) — Bump the `version` field in `package.json`
- [C# (.csproj)](examples/csproj.yml) — Chain-bump `Version` and `FileVersion`

## How It Works

1. Validates input parameters
2. Installs the bump CLI via `go install github.com/mattn/bump@latest`
3. Runs `bump` with the `-w` flag (writes the file in-place and prints the new version to stdout)
4. Sets the new version as the action output

## Requirements

- GitHub-hosted runners have Go pre-installed, so no additional setup is needed
- For self-hosted runners, Go must be installed

## License

MIT
