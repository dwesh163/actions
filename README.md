# actions

Reusable GitHub Actions for dwesh163 projects.

## detect-version

Detects the project version and checks if a release already exists.

```yaml
- uses: dwesh163/actions/detect-version@main
  with:
    type: python        # js or python
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

**Inputs**

| Name | Required | Description |
|------|----------|-------------|
| `type` | yes | `js` (reads `package.json`) or `python` (reads `setup.py`) |
| `github-token` | yes | Token used to check existing releases |

**Outputs**

| Name | Description |
|------|-------------|
| `version` | Detected version string |
| `release_exist` | `true` if a release already exists for this version |

---

## release

Generates release notes from git history and creates a GitHub release.

> Requires `fetch-depth: 0` on the preceding `actions/checkout` step.

```yaml
- uses: dwesh163/actions/release@main
  with:
    version: ${{ steps.detect.outputs.version }}
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

**Inputs**

| Name | Required | Description |
|------|----------|-------------|
| `version` | yes | Version to release (without `v` prefix) |
| `github-token` | yes | Token used to create the release |

Release notes are auto-generated from commits since the last tag, grouped by prefix: `[feat]`, `[fix]`, `[refactor]`, `[docs]`, etc.

---

## Usage example

```yaml
detect-version:
  runs-on: ubuntu-24.04
  permissions:
    contents: read
  outputs:
    version: ${{ steps.detect.outputs.version }}
    release_exist: ${{ steps.detect.outputs.release_exist }}
  steps:
    - uses: actions/checkout@v4
    - id: detect
      uses: dwesh163/actions/detect-version@main
      with:
        type: python
        github-token: ${{ secrets.GITHUB_TOKEN }}

create-release:
  needs: [detect-version]
  if: needs.detect-version.outputs.release_exist == 'false'
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
