# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [Unreleased]

### Added
- `.github/workflows/mirror-to-gitlab.yml`: Reusable GitHub Actions workflow (`workflow_call`) to synchronize all branches and tags to a target GitLab clone mirror with secret sharing (`GITLAB_TOKEN`), repository variable fallback (`vars.GITLAB_PROJECT_PATH`), and scoped concurrency.
- `.github/workflows/dependabot-auto-merge.yml`: Reusable GitHub Actions workflow (`workflow_call`) to evaluate, approve, and automatically merge trusted Dependabot pull requests based on semantic version updates.

### Changed
- `README.md`: Added catalog entry, zero-YAML configuration guide via repository variables, and inline input examples for GitLab mirroring.
- `CONTRIBUTING.md`: Documented input/secret casing conventions, repository variable fallback patterns, and repository-scoped concurrency best practices.

### Planned
- Reusable workflow for Rust CI (`ci-rust.yml`) with cargo-audit and cross-compilation support.
- Automated Slack/Discord notification reusable workflow on pipeline failure.

---

## [1.0.0] - 2026-10-25

### Added
- **Repository Architecture**: Initial project setup designed for centralized, reusable GitHub Actions workflows (`workflow_call`).
- **Comprehensive Documentation**:
  - [`README.md`](README.md): Detailed usage guide, workflow catalog, versioning strategies (`@v1` vs SHA pinning), and multi-stage workflow integration examples.
  - [`SECURITY.md`](SECURITY.md): Vulnerability reporting procedures, response SLAs, contact details, and consumer hardening recommendations.
  - [`CONTRIBUTING.md`](CONTRIBUTING.md): Contribution guidelines, `workflow_call` coding standards, injection protection rules, Conventional Commits specification, and local testing instructions with `actionlint`.
  - [`CHANGELOG.md`](CHANGELOG.md): Standardized version tracking based on Keep a Changelog.
- **Workflow Catalog Design**:
  - `lint.yml`: Reusable multi-language linting and code quality checks.
  - `ci-node.yml`: Reusable Node.js matrix test runner with caching support (npm, yarn, pnpm, bun).
  - `ci-python.yml`: Reusable Python test runner supporting pytest, coverage, and modern package managers.
  - `ci-go.yml`: Reusable Go test and build pipeline with race detection and linting.
  - `docker-publish.yml`: Reusable container image builder with Buildx and registry push (GHCR / Docker Hub).
  - `security-scan.yml`: Reusable vulnerability audit using Gitleaks and Trivy.
  - `release.yml`: Reusable semantic release automation and changelog drafting.
  - `actionlint.yml`: Internal CI workflow to validate workflow syntax and security.

[Unreleased]: https://github.com/fapethedev/workflows/compare/v1.0.0...HEAD
[1.0.0]: https://github.com/fapethedev/workflows/releases/tag/v1.0.0
