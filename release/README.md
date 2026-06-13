# release

Generates release notes from git history and publishes a GitHub release.

Commits are grouped automatically by prefix: `[feat]`, `[fix]`, `[refactor]`, `[docs]`, and others.

> Requires `fetch-depth: 0` on the preceding `actions/checkout` step to access full git history.

## Usage

```yaml
- uses: dwesh163/actions/release@main
  with:
    version: "1.2.3"
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

## Inputs

| Name | Required | Description |
|------|----------|-------------|
| `version` | yes | Version to release, without the `v` prefix |
| `github-token` | yes | Token used to create the GitHub release |

## Example

```yaml
jobs:
  create-release:
    runs-on: ubuntu-24.04
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: dwesh163/actions/release@main
        with:
          version: ${{ needs.detect-version.outputs.version }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
```
