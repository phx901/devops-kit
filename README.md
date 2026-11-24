# devops-kit

[![DevOps Kit CI](https://github.com/phx901/devops-kit/actions/workflows/ci.yml/badge.svg)](https://github.com/phx901/devops-kit/actions/workflows/ci.yml)

Central repository for shared GitHub Actions workflows and configuration.

## Reusable Workflows

The following workflows can be integrated into your projects for common CI/CD tasks.

### Version Calculation — `semver-create.yml`

Calculates the semantic version using [GitVersion](https://gitversion.net/) and the `GitVersion.yaml` config from this repo.

**Outputs:** `version` (SemVer, e.g. `1.2.3` on `main`, `1.3.0-alpha.4` on other branches)

```yaml
jobs:
  versioning:
    uses: phx901/devops-kit/.github/workflows/semver-create.yml@main
```

---

### Version Tagging — `semver-tag.yml`

Creates and pushes a `vX.Y.Z` tag to the repository.

```yaml
jobs:
  tagging:
    uses: phx901/devops-kit/.github/workflows/semver-tag.yml@main
    permissions:
      contents: write
    with:
      version: ${{ needs.versioning.outputs.version }}
```

**Inputs:**

| Input | Required | Default | Description |
|---|---|---|---|
| `version` | ✅ | – | Version to tag, e.g. `1.2.3` |

---

### Node.js Build & Test — `nodejs-build.yml`

Installs dependencies, runs tests, builds a Node.js project and optionally bundles production dependencies into the build output and uploads it as a GitHub Artifact.

```yaml
jobs:
  build:
    uses: phx901/devops-kit/.github/workflows/nodejs-build.yml@main
    with:
      working-directory: ./my-api
```

```yaml
jobs:
  build:
    uses: phx901/devops-kit/.github/workflows/nodejs-build.yml@main
    with:
      working-directory: ./my-api
      install-production-dependencies: true
      upload-artifact: true
      artifact-name: nodejs-dist
```

**Inputs:**

| Input | Required | Default | Description |
|---|---|---|---|
| `working-directory` | ✅ | – | Directory containing the Node.js project |
| `test-args` | – | `""` | Extra arguments to pass to npm test (e.g. `--no-watch`, `--run`) |
| `build-output-path` | – | `dist` | Build output folder relative to `working-directory` |
| `install-production-dependencies` | – | `false` | Copy `package.json` and `package-lock.json` into the build output and install production dependencies there |
| `upload-artifact` | – | `false` | Upload the build output as a GitHub Artifact |
| `artifact-name` | ✅ (when `upload-artifact` is `true`) | `""` | Name for the uploaded artifact |

---

### Angular Build & Test — `angular-build.yml`

Installs dependencies, runs tests, injects the version into `version.ts`, builds the Angular project and uploads the `dist/` folder as a GitHub Artifact.

```yaml
jobs:
  build:
    uses: phx901/devops-kit/.github/workflows/angular-build.yml@main
    with:
      working-directory: ./my-angular-app
      version: ${{ needs.versioning.outputs.version }}
      upload-artifact: true
      artifact-name: my-app-dist
```

**Inputs:**

| Input | Required | Default | Description |
|---|---|---|---|
| `working-directory` | ✅ | – | Directory containing the Angular project |
| `version` | – | – | Version to inject into `version.ts` |
| `version-file-path` | – | `src/environments/version.ts` | Path to the version file (relative to `working-directory`) |
| `test-args` | – | `""` | Extra arguments to pass to npm test (e.g. `--no-watch`) |
| `upload-artifact` | – | `false` | Upload the `dist/` folder as a GitHub Artifact |
| `artifact-name` | ✅ (when `upload-artifact` is `true`) | `""` | Name for the uploaded artifact |

---

### .NET Build & Test — `dotnet-build.yml`

Restores dependencies, runs tests (with TRX report), builds, and optionally publishes a .NET project and uploads the output as a GitHub Artifact.

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

**Inputs:**

| Input | Required | Default | Description |
|---|---|---|---|
| `working-directory` | ✅ | – | Directory containing the .NET project |
| `publish-project` | – | – | Project file to publish (e.g. `src/MyApi/MyApi.csproj`) |
| `runtime` | – | `linux-arm64` | Publish runtime |
| `upload-artifact` | – | `false` | Upload the publish output as a GitHub Artifact |
| `artifact-name` | ✅ (when `upload-artifact` is `true`) | `""` | Name for the uploaded artifact |

---

### Angular Deploy to AWS — `angular-deploy-aws.yml`

Downloads a build artifact and deploys it to an S3 bucket, then invalidates a CloudFront distribution.

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

**Inputs:**

| Input | Required | Default | Description |
|---|---|---|---|
| `artifact-name` | ✅ | – | Name of the artifact uploaded by the build job |
| `dist-path` | ✅ | – | Path to the `dist` folder inside the artifact (e.g. `my-app/browser`) |
| `s3-bucket` | ✅ | – | S3 bucket to sync to |
| `cloudfront-distribution-id` | ✅ | – | CloudFront distribution to invalidate |
| `aws-region` | ✅ | – | AWS region |

**Secrets:**

| Secret | Required |
|---|---|
| `aws-access-key-id` | ✅ |
| `aws-secret-access-key` | ✅ |

---

### Deploy to AWS Lambda — `aws-lambda-deploy.yml`

Downloads a build artifact, packages it as a ZIP, deploys it to an AWS Lambda function, and optionally sets the `AppVersion` Lambda environment variable. Works for any Lambda runtime whose artifact can be zipped as-is (e.g. Node.js artifacts from `nodejs-build.yml`).

```yaml
jobs:
  deploy:
    uses: phx901/devops-kit/.github/workflows/aws-lambda-deploy.yml@main
    with:
      artifact-name: nodejs-dist
      lambda-function-name: my-lambda-function
      aws-region: eu-central-1
      version: ${{ needs.versioning.outputs.version }}
    secrets:
      aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
      aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

**Inputs:**

| Input | Required | Default | Description |
|---|---|---|---|
| `artifact-name` | ✅ | – | Name of the artifact uploaded by the build job |
| `lambda-function-name` | ✅ | – | Name of the Lambda function to update |
| `aws-region` | ✅ | – | AWS region |
| `version` | – | `""` | Application version to set as Lambda environment variable `AppVersion` (e.g. `1.2.3`) |

**Secrets:**

| Secret | Required |
|---|---|
| `aws-access-key-id` | ✅ |
| `aws-secret-access-key` | ✅ |

## Branching Strategy

[GitHub Flow](https://docs.github.com/en/get-started/using-github/github-flow): short-lived branches, merged into `main` via pull request.

Versioning is managed via `GitVersion.yaml`:

| Branch pattern                                       | Mode                 | Increment     | Label   | Example version |
|------------------------------------------------------|----------------------|---------------|---------|-----------------|
| `main`                                               | ContinuousDeployment | Merged branch | —       | `9.0.1`         |
| `feature/*`                                          | ContinuousDelivery   | Minor         | `alpha` | `9.1.0-alpha.4` |
| `patch/*` / `hotfix/*` / `bugfix/*` / `dependabot/*` | ContinuousDelivery   | Patch         | `alpha` | `9.0.2-alpha.2` |
| Pull request                                         | ContinuousDelivery   | Source branch | `PullRequest<n>` | `9.0.2-PullRequest12.1` |

Tags are prefixed with `v` (e.g. `v1.2.3`).

## Maintenance

Dependabot updates GitHub Actions dependencies weekly (Mondays, 05:00 CET).
