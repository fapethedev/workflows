# Security Policy

The security of our automation infrastructure and the repositories consuming these reusable workflows is of paramount importance. This document outlines our security policies, supported versions, and procedures for reporting security vulnerabilities.

---

## Supported Versions

We provide security patches and updates for the active release branches. We encourage all consumer repositories to stay pinned to either the latest major version tag (`@v1`) or explicit commit SHAs.

| Version | Supported          | Release Status |
| :------ | :----------------- | :------------- |
| `v1.x`  | :white_check_mark: | Active         |
| `< v1.0`| :x:                | End of Life    |

---

## Reporting a Vulnerability

If you discover a security vulnerability, security flaw, or potential supply chain exposure in any workflow in this repository:

> [!CAUTION]
> **Do not open a public GitHub issue, discussion, or pull request for security vulnerabilities.**

Please report security issues using one of the following methods:

### Method 1: GitHub Private Vulnerability Reporting (Preferred)
Navigate to the repository's **Security** tab, click on **Advisories**, and select **Report a vulnerability**. This creates a confidential disclosure channel directly with the maintainers.

### Method 2: Direct Contact via Email
Send an encrypted or confidential email to:
- **Email**: `mpetrivlin@gmail.com`
- **Subject**: `[SECURITY VULNERABILITY] workflows - <Short Description>`

### Details to Include in Your Report
To help us triage and remediate the issue promptly, please include:
1. **Affected Workflow**: File path and version/commit SHA (e.g., `.github/workflows/docker-publish.yml@v1`).
2. **Vulnerability Type**: e.g., Script Injection, Insecure Secret Handling, Excessive Permissions, Insecure Third-Party Action.
3. **Proof of Concept (PoC)**: Minimal reproducing workflow YAML or steps to trigger the vulnerability.
4. **Impact Assessment**: What an attacker could gain (e.g., repository write access, secret exfiltration, malicious artifact injection).
5. **Mitigation Suggestion**: Any proposed fix or remediation code (if available).

---

## Response Process & SLAs

1. **Initial Acknowledgment**: We will acknowledge receipt of your vulnerability report within **48 hours**.
2. **Triage & Validation**: Maintainers will analyze the report, confirm reproducibility, and evaluate severity within **5 business days**.
3. **Patch Development & Testing**: Fixes will be prepared in private repositories or security advisory workspaces.
4. **Coordinated Disclosure**: Once patched, a new release and a public GitHub Security Advisory (with credit to the reporter, if desired) will be published.

---

## Security Hardening Guidelines for Consumers

When consuming reusable workflows from this repository, adhere to the following best practices in your caller repositories:

### 1. Enforce Principle of Least Privilege (`permissions`)
Always restrict `GITHUB_TOKEN` permissions at both the workflow and job level. Do not grant broader permissions than required:

```yaml
# Recommended: Default to read-only or empty permissions
permissions:
  contents: read
```

### 2. Pin to Immutable Commit SHAs
To protect against upstream tag manipulation and supply chain compromise, reference reusable workflows by their immutable 40-character commit SHA:

```yaml
# Recommended for mission-critical pipelines:
uses: fapethedev/workflows/.github/workflows/ci-node.yml@8f2d59e3cf4d8b9bb2e153b0a72ad9b7941ceb61 # v1.2.0
```

Enable [Dependabot](https://docs.github.com/en/code-security/dependabot) in `.github/dependabot.yml` to automatically receive pull requests when new versions are released.

### 3. Handle Untrusted Pull Requests Securely
- Never trigger reusable workflows using `pull_request_target` unless code checkout is strictly pinned to the base branch (`main` / `master`) and does not build or run code from the untrusted fork.
- Do not expose repository secrets to pull requests coming from external forks.

### 4. Explicit Secret Forwarding
Avoid blanket `secrets: inherit`. Instead, explicitly declare only the secrets required by the invoked workflow:

```yaml
# Recommended:
secrets:
  registry-token: ${{ secrets.CONTAINER_REGISTRY_TOKEN }}
```

---

## Internal Security Measures

Workflows maintained in this repository are subject to continuous automated checks:
- **Linting & AST Inspection**: Workflows are verified via `actionlint` to eliminate syntax errors, unquoted expressions, and command injection risks.
- **Secret Detection**: Pre-commit and CI scans using tools like Gitleaks to prevent accidental credential commits.
- **Container & Dependency Auditing**: Vulnerability scans executed with Trivy to identify CVEs in base images and toolchains.
