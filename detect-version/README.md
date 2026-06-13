# detect-version

Detects the project version from `package.json` (JS) or `setup.py` (Python) and checks whether a GitHub release already exists for that version.

## Usage

```yaml
- uses: dwesh163/actions/detect-version@main
  with:
    type: python
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

## Inputs

| Name | Required | Values | Description |
|------|----------|--------|-------------|
| `type` | yes | `js`, `python` | Source file to read the version from |
| `github-token` | yes | | Token used to query existing releases |

## Outputs

| Name | Description |
|------|-------------|
| `version` | Detected version string (e.g. `1.2.3`) |
| `release_exist` | `true` if a release already exists for this version, `false` otherwise |

## Example

```yaml
jobs:
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
```
