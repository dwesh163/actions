# bump-chart

Bumps the `appVersion` (and optionally the `version`) of a Helm chart living in a separate charts repository, and pushes the change as a commit `[auto] Bump <chart-name> chart to version <version>`.

Authentication is done via a GitHub App (not `GITHUB_TOKEN`), since the target repository is usually different from the one running the workflow. The app must be installed on the chart repository with **Contents: Read and write** permission.

## Usage

```yaml
- uses: dwesh163/actions/bump-chart@main
  with:
    chart-repo: xdew-cloud/charts
    chart-name: my-app
    app-version: ${{ github.event.release.tag_name }}
    github-app-id: ${{ secrets.GH_APP_ID }}
    github-app-private-key: ${{ secrets.GH_APP_SECRET }}
```

## Inputs

| Name | Required | Default | Description |
|------|----------|---------|-------------|
| `chart-repo` | yes | | Target chart repository, e.g. `xdew-cloud/charts` |
| `chart-name` | yes | | Name of the chart/app to bump, used as the default folder name and in the commit message |
| `app-version` | yes | | New `appVersion` to set (a leading `v` is stripped automatically) |
| `chart-path` | no | `<chart-name>` | Path to the chart directory inside `chart-repo`, if not the default layout |
| `bump-chart-version` | no | `true` | Also bump the patch segment of the Chart.yaml `version` field |
| `branch` | no | repo default branch | Branch to checkout and push to in `chart-repo` |
| `github-app-id` | yes | | GitHub App ID (`GH_APP_ID`) used to authenticate the commit |
| `github-app-private-key` | yes | | GitHub App private key (`GH_APP_SECRET`) used to authenticate the commit |

## Outputs

| Name | Description |
|------|-------------|
| `committed` | `true` if a commit was pushed, `false` if the chart was already at the target `appVersion` |
| `commit-sha` | SHA of the created commit, if any |
| `chart-version` | Resulting Chart.yaml `version` field after the bump |

## Example

```yaml
jobs:
  bump-chart:
    runs-on: ubuntu-24.04
    steps:
      - uses: dwesh163/actions/bump-chart@main
        with:
          chart-repo: xdew-cloud/charts
          chart-name: api
          app-version: ${{ needs.detect-version.outputs.version }}
          github-app-id: ${{ secrets.GH_APP_ID }}
          github-app-private-key: ${{ secrets.GH_APP_SECRET }}
```
