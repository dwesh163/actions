# Reusable GitHub Actions

A collection of reusable GitHub Actions to standardize version detection and release automation across projects.

## Actions

- [detect-version](./detect-version) — Detect version from `package.json` or `setup.py` and check if a release already exists
- [release](./release) — Generate release notes from git history and publish a GitHub release
- [bump-chart](./bump-chart) — Bump a Helm chart's `appVersion` in a separate charts repo and commit as a GitHub App

## Reusable workflows

The pull request and release pipelines of a Docker app are the same in every repository. They are two reusable workflows here, so a repository keeps only what is specific to it: its lint and a short call.

| Workflow | Does |
|---|---|
| [docker-pr](.github/workflows/docker-pr.yaml) | Builds the image of a pull request and pushes `ghcr.io/<repository>:<branch>-<sha>`. Skipped for pull requests from forks. |
| [docker-release](.github/workflows/docker-release.yaml) | When the version is new: builds and pushes `:latest` and `:<version>`, creates the GitHub release, and bumps a chart if `chart-repo` and `chart-name` are set. |

A repository keeps three files. `_lint.yaml` stays local because lint depends on the language (`needs: lint` makes the shared job wait for it).

```yaml
# .github/workflows/release.yaml
name: Release

on:
  push:
    branches: [main]

permissions: {}

concurrency:
  group: release
  cancel-in-progress: false

jobs:
  lint:
    permissions:
      contents: read
    uses: ./.github/workflows/_lint.yaml

  release:
    needs: lint
    permissions:
      contents: write
      packages: write
    uses: dwesh163/actions/.github/workflows/docker-release.yaml@<sha> # <tag>
    with:
      version-type: js            # js | python | file
      chart-repo: xdew-cloud/charts
      chart-name: console
    secrets:
      GH_APP_ID: ${{ secrets.GH_APP_ID }}
      GH_APP_SECRET: ${{ secrets.GH_APP_SECRET }}
```

```yaml
# .github/workflows/pr.yaml
name: Pull Request

on:
  pull_request:
    branches: [main]

permissions: {}

concurrency:
  group: pr-${{ github.ref }}
  cancel-in-progress: true

jobs:
  lint:
    permissions:
      contents: read
    uses: ./.github/workflows/_lint.yaml

  image:
    needs: lint
    permissions:
      contents: read
      packages: write
    uses: dwesh163/actions/.github/workflows/docker-pr.yaml@<sha> # <tag>
```

Inputs, all optional: `set-version` (`package-json` default, `file` writes `.version`, `none`), `build-args`, `context`, `dockerfile`; for the release also `version-type`, `version-path`, `chart-repo`, `chart-name`. Repositories that build several images, or not with Docker (Go services with their own build), keep their own `_build.yaml`.

The two workflows repeat the build steps (about 20 lines): a shared composite action would have to be pinned to a commit that does not exist yet when it is added. Pin the workflows to a commit or a tag, never to `main`.
