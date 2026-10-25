# Reusable GitHub Actions Workflows

[![GitHub License](https://img.shields.io/github/license/fapethedev/workflows)](LICENSE)
[![GitHub release](https://img.shields.io/github/v/release/fapethedev/workflows?include_prereleases&sort=semver)](https://github.com/fapethedev/workflows/releases)
[![Action Lint](https://img.shields.io/badge/actionlint-passing-brightgreen)](https://github.com/rhysd/actionlint)
[![Security Policy](https://img.shields.io/badge/security-enforced-blue)](SECURITY.md)

A standardized collection of production-grade, secure, and reusable [GitHub Actions workflows](https://docs.github.com/en/actions/sharing-automations/reusing-workflows) designed to simplify CI/CD pipelines, code quality assurance, security auditing, and release management across your GitHub repositories.

---

## Overview

Rather than duplicating complex workflow YAML files across multiple projects, this repository acts as a single source of truth. Repositories consume these workflows using GitHub Actions' native `workflow_call` trigger, ensuring:

- **Consistency**: Standardized linting, testing, security, and packaging across all projects.
- **Security by Default**: Least-privilege permissions, SHA pinning compliance, and automated vulnerability scanning.
- **Maintainability**: Fix bugs or upgrade action versions centrally without touching individual project repositories.
- **Faster Setup**: Bootstrap full CI/CD pipelines in new repositories in minutes.

---

## Quick Start

### Basic Usage Syntax

Reference any reusable workflow in this repository using the `uses` keyword inside a job:

```yaml
# .github/workflows/ci.yml in your consumer repository
name: CI

on:
  push:
    branches: [main, master]
  pull_request:
    branches: [main, master]

jobs:
  lint:
    name: Code Quality
    uses: fapethedev/workflows/.github/workflows/lint.yml@v1
    with:
      linter: biome
```

> [!TIP]
> **Version Pinning Best Practice**:
> - Use major version tags (`@v1`) for automatic minor and patch updates.
> - For maximum security and reproducible builds, pin to a full 40-character commit SHA (e.g. `@d5c21...`) and let [Dependabot](https://docs.github.com/en/code-security/dependabot) manage workflow updates.

---

## Workflow Catalog

| Workflow | Path | Description | Triggers Supported |
| :--- | :--- | :--- | :--- |
| **Lint & Format** | [`.github/workflows/lint.yml`](.github/workflows/lint.yml) | Multi-language code quality and style checks (ESLint, Biome, Prettier, Ruff, GoFmt). | `workflow_call` |
| **Node CI** | [`.github/workflows/ci-node.yml`](.github/workflows/ci-node.yml) | Node.js testing, matrix execution (Node 18/20/22), dependency caching (npm/yarn/pnpm/bun). | `workflow_call` |
| **Python CI** | [`.github/workflows/ci-python.yml`](.github/workflows/ci-python.yml) | Python testing with pytest, coverage upload, poetry/uv/pip support. | `workflow_call` |
| **Go CI** | [`.github/workflows/ci-go.yml`](.github/workflows/ci-go.yml) | Go build, test with race detection, golangci-lint, and coverage analysis. | `workflow_call` |
| **Docker Build & Push** | [`.github/workflows/docker-publish.yml`](.github/workflows/docker-publish.yml) | Multi-platform container build using Buildx, caching, and publishing to GHCR or Docker Hub. | `workflow_call` |
| **Security Audit** | [`.github/workflows/security-scan.yml`](.github/workflows/security-scan.yml) | Vulnerability scanning (Trivy), secret leak detection (Gitleaks), and dependency review. | `workflow_call` |
| **Release & Changelog** | [`.github/workflows/release.yml`](.github/workflows/release.yml) | Automated semantic version tagging, GitHub release generation, and changelog updates. | `workflow_call` |
| **GitLab Mirror** | [`.github/workflows/mirror-to-gitlab.yml`](.github/workflows/mirror-to-gitlab.yml) | Synchronizes all branches and tags from GitHub to a target GitLab project mirror. | `workflow_call`, `workflow_dispatch` |
| **Dependabot Auto-Merge** | [`.github/workflows/dependabot-auto-merge.yml`](.github/workflows/dependabot-auto-merge.yml) | Validates, approves, and automatically merges trusted Dependabot PRs. | `workflow_call` |
| **Actionlint & Self-Check** | [`.github/workflows/actionlint.yml`](.github/workflows/actionlint.yml) | Linting and validation of GitHub Actions YAML workflows. | `workflow_call`, `push`, `pull_request` |

---

## Usage Examples

### 1. Multi-Stage Pipeline with Node.js & Security Scanning

```yaml
name: Continuous Integration

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

permissions:
  contents: read

jobs:
  security:
    name: Security Scans
    uses: fapethedev/workflows/.github/workflows/security-scan.yml@v1
    with:
      scan-secrets: true
      scan-dependencies: true

  quality:
    name: Quality Checks
    uses: fapethedev/workflows/.github/workflows/lint.yml@v1
    with:
      linter: eslint

  test:
    name: Automated Tests
    needs: [security, quality]
    uses: fapethedev/workflows/.github/workflows/ci-node.yml@v1
    with:
      node-version: '20.x'
      package-manager: 'pnpm'
      run-tests: true
```

### 2. Building & Publishing a Multi-Arch Docker Image

```yaml
name: Build & Push Container

on:
  push:
    tags:
      - 'v*.*.*'

permissions:
  contents: read
  packages: write

jobs:
  publish-image:
    name: Docker Publish
    uses: fapethedev/workflows/.github/workflows/docker-publish.yml@v1
    with:
      registry: ghcr.io
      image-name: ${{ github.repository }}
      platforms: linux/amd64,linux/arm64
    secrets:
      registry-token: ${{ secrets.GITHUB_TOKEN }}
```

### 3. Continuous Mirroring to GitLab

Automatically sync all branches and tags from any GitHub repository to a target GitLab project.

#### Option A: Zero plain-text configuration (Recommended)
Configure `GITLAB_PROJECT_PATH` in GitHub repository variables (**Settings > Secrets and variables > Actions > Variables**) and `GITLAB_TOKEN` in secrets. The caller YAML stays completely clean without exposing project paths in code:

```yaml
name: Mirror to GitLab

on:
  push:
    branches:
      - '**'
    tags:
      - '**'
  workflow_dispatch:

jobs:
  mirror:
    name: Sync to GitLab
    uses: fapethedev/workflows/.github/workflows/mirror-to-gitlab.yml@master
    secrets: inherit
```

#### Option B: Inline configuration with explicit input
You can also specify or override the target path directly via `with.gitlab_project_path`:

```yaml
name: Mirror to GitLab

on:
  push:
    branches:
      - '**'
    tags:
      - '**'
  workflow_dispatch:

jobs:
  mirror:
    name: Sync to GitLab
    uses: fapethedev/workflows/.github/workflows/mirror-to-gitlab.yml@master
    with:
      gitlab_project_path: 'fapethedev/my-target-project'
    secrets:
      GITLAB_TOKEN: ${{ secrets.GITLAB_TOKEN }}
```

> [!TIP]
> **Resolution Priority**:
> 1. `inputs.gitlab_project_path` (if supplied in YAML)
> 2. `vars.GITLAB_PROJECT_PATH` (configured in GitHub repository / organization variables)
> 3. `github.repository` (default fallback when project names match)

### 4. Dependabot Auto-Merge

Automatically approve and enable auto-merge on trusted Dependabot Pull Requests (patch and minor updates):

```yaml
# .github/workflows/dependabot-automerge.yml in consumer repo
name: Dependabot Auto-Merge

on:
  pull_request:

permissions:
  contents: write
  pull-requests: write

jobs:
  auto-merge:
    name: Auto-Merge Trusted PRs
    uses: fapethedev/workflows/.github/workflows/dependabot-auto-merge.yml@master
    secrets: inherit
```

> [!NOTE]
> **Required Repository Settings**:
> 1. Under **Settings > General > Pull Requests**, enable **Allow auto-merge**.
> 2. Under **Settings > Actions > General > Workflow permissions**, enable **Allow GitHub Actions to create and approve pull requests**.

### 5. Passing Secrets Securely

Reusable workflows allow passing explicit secrets or inheriting caller repository secrets:

```yaml
jobs:
  deploy:
    uses: fapethedev/workflows/.github/workflows/deploy.yml@v1
    with:
      environment: production
    secrets:
      api-token: ${{ secrets.PROD_API_TOKEN }}
      ssh-key: ${{ secrets.PROD_SSH_KEY }}
```

> [!IMPORTANT]
> To comply with the principle of least privilege, avoid passing `secrets: inherit` unless strictly required. Explicitly bind required secrets as shown above.

---

## Permissions & Hardening

All workflows in this repository are engineered following security best practices:

- **Minimal Permissions by Default**: Workflows specify top-level `permissions: {}` or `permissions: contents: read` unless specific elevated rights (e.g. `packages: write`, `id-token: write`) are required by the target job.
- **Step-Level Token Scoping**: Third-party actions cannot access secrets not explicitly passed to them.
- **Injection Protection**: Direct shell evaluation of untrusted input expressions (like `${{ github.event.issue.title }}`) is strictly prohibited; inputs are mediated through environment variables.

---

## Directory Structure

```text
.
├── .github/
│   └── workflows/
│       ├── actionlint.yml          # Lints all workflows in this repository
│       ├── ci-go.yml               # Reusable Go test and lint pipeline
│       ├── ci-node.yml             # Reusable Node.js test pipeline
│       ├── ci-python.yml           # Reusable Python test pipeline
│       ├── docker-publish.yml      # Reusable multi-arch container pipeline
│       ├── lint.yml                # Generic code linter
│       ├── release.yml             # Reusable release tagger & notes generator
│       └── security-scan.yml       # Reusable Gitleaks & Trivy security audit
├── CONTRIBUTING.md                 # Contribution guide & workflow guidelines
├── SECURITY.md                     # Security policies & vulnerability reporting
├── CHANGELOG.md                    # Keep a Changelog standard history
└── README.md                       # Project documentation
```

---

## Verification & Local Testing

Before invoking these workflows or committing new reusable definitions, you can lint and test them locally:

- **Lint workflows with `actionlint`**:
  ```bash
  actionlint
  ```

- **Run workflows locally with `act`**:
  ```bash
  act -l
  ```
