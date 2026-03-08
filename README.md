# action-bump-cli

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

```yaml
- uses: supermomonga/action-bump-cli@v1
  id: bump
  with:
    file: package.json
    pattern: '"version":\s*"(\d+\.\d+\.\d+)"'
    release_type: minor

- run: echo "New version: ${{ steps.bump.outputs.version }}"
```

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
- [C# (.csproj)](examples/csproj.yml) — Chain-bump `Version`, `FileVersion`, and `AssemblyVersion`

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
