# devops-kit

Shared GitHub Actions workflows and configuration used across `adanze` repositories.

## Reusable Workflows

### Version Calculation — `semver-create.yml`

Calculates the semantic version using [GitVersion](https://gitversion.net/) and the `GitVersion.yaml` config from this repo.

**Outputs:** `version` (e.g. `1.2.3`), `semver` (e.g. `1.2.3-alpha.4`)

```yaml
jobs:
  versioning:
    uses: adanze/devops-kit/.github/workflows/semver-create.yml@main
```

---

### Git Tagging — `semver-tag.yml`

Creates and pushes a `vX.Y.Z` tag to the repository.

**Inputs:** `version` (required, e.g. `1.2.3`)

```yaml
jobs:
  tagging:
    uses: adanze/devops-kit/.github/workflows/semver-tag.yml@main
    permissions:
      contents: write
    with:
      version: ${{ needs.versioning.outputs.version }}
```

---

### Node.js Build & Test — `nodejs-build.yml`

Installs dependencies, runs tests, and builds a Node.js project.

**Inputs:** `working-directory` (required), `test-args` (optional)

```yaml
jobs:
  build:
    uses: adanze/devops-kit/.github/workflows/nodejs-build.yml@main
    with:
      working-directory: ./my-app
```

## Branching Strategy

Managed via `GitVersion.yaml` in `ContinuousDelivery` mode:

| Branch pattern      | Increment | Label   |
|---------------------|-----------|---------|
| `main`              | None      | —       |
| `develop`           | None      | `beta`  |
| `feature/*`         | Minor     | `alpha` |
| `patch/*` / `hotfix/*` / `bugfix/*` | Patch | `alpha` |

Tags are prefixed with `v` (e.g. `v1.2.3`).

## Maintenance

Dependabot updates GitHub Actions dependencies weekly (Mondays, 05:00 CET).
