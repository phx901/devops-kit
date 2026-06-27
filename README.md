# devops-kit

[![DevOps Kit CI](https://github.com/phx901/devops-kit/actions/workflows/ci.yml/badge.svg)](https://github.com/phx901/devops-kit/actions/workflows/ci.yml)

Central repository for shared GitHub Actions workflows and configuration.

## Reusable Workflows

The following workflows can be integrated into your projects for common CI/CD tasks.

### Version Calculation — `semver-create.yml`

Calculates the semantic version using [GitVersion](https://gitversion.net/) and the `GitVersion.yaml` config from this repo.

**Outputs:** `version` (e.g. `1.2.3`), `semver` (e.g. `1.2.3-alpha.4`)

```yaml
jobs:
  versioning:
    uses: phx901/devops-kit/.github/workflows/semver-create.yml@main
```

---

### Git Tagging — `semver-tag.yml`

Creates and pushes a `vX.Y.Z` tag to the repository.

**Inputs:** `version` (required, e.g. `1.2.3`)

```yaml
jobs:
  tagging:
    uses: phx901/devops-kit/.github/workflows/semver-tag.yml@main
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
    uses: phx901/devops-kit/.github/workflows/nodejs-build.yml@main
    with:
      working-directory: ./my-api
```

---

### Angular Build & Test — `angular-build.yml`

Installs dependencies, runs tests, injects the version into `version.ts`, builds the Angular project and uploads the `dist/` folder as a GitHub Artifact.

**Inputs:** `working-directory` (required), `version` (optional), `version-file-path` (optional, default: `src/environments/version.ts`), `test-args` (optional), `upload-artifact` (optional, default: `false`), `artifact-name` (required when `upload-artifact` is `true`)

```yaml
jobs:
  build:
    uses: phx901/devops-kit/.github/workflows/angular-build.yml@main
    with:
      working-directory: ./my-angular-app
      version: ${{ needs.versioning.outputs.semver }}
      upload-artifact: true
      artifact-name: my-app-dist
```

---

### .NET Build & Test — `dotnet-build.yml`

Restores dependencies, runs tests (with TRX report), builds, and optionally publishes a .NET project and uploads the output as a GitHub Artifact.

**Inputs:** `working-directory` (required), `publish-project` (optional), `runtime` (optional, default: `linux-arm64`), `upload-artifact` (optional, default: `false`), `artifact-name` (required when `upload-artifact` is `true`)

```yaml
jobs:
  build:
    uses: phx901/devops-kit/.github/workflows/dotnet-build.yml@main
    with:
      working-directory: ./my-dotnet-api
      publish-project: src/MyApi/MyApi.csproj
      upload-artifact: true
      artifact-name: dotnet-dist
```

---

### Angular Deploy to AWS — `angular-deploy-aws.yml`

Downloads a build artifact and deploys it to an S3 bucket, then invalidates a CloudFront distribution.

**Inputs:** `artifact-name` (required), `dist-path` (required), `s3-bucket` (required), `cloudfront-distribution-id` (required), `aws-region` (required)

**Secrets:** `aws-access-key-id` (required), `aws-secret-access-key` (required)

```yaml
jobs:
  deploy:
    uses: phx901/devops-kit/.github/workflows/angular-deploy-aws.yml@main
    with:
      artifact-name: my-app-dist
      dist-path: my-app/browser
      s3-bucket: my-bucket
      cloudfront-distribution-id: ABCDEF123456
      aws-region: eu-central-1
    secrets:
      aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
      aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

---

### .NET Deploy to AWS Lambda — `dotnet-deploy-aws-lambda.yml`

Downloads a build artifact, packages it as a ZIP, deploys it to an AWS Lambda function, and optionally sets the `AppVersion` Lambda environment variable.

**Inputs:** `artifact-name` (required), `lambda-function-name` (required), `aws-region` (required), `version` (optional)

**Secrets:** `aws-access-key-id` (required), `aws-secret-access-key` (required)

```yaml
jobs:
  deploy:
    uses: phx901/devops-kit/.github/workflows/dotnet-deploy-aws-lambda.yml@main
    with:
      artifact-name: dotnet-dist
      lambda-function-name: my-lambda-function
      aws-region: eu-central-1
      version: ${{ needs.versioning.outputs.version }}
    secrets:
      aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
      aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
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
