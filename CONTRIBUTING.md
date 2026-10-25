# Contributing to Reusable GitHub Actions Workflows

Thank you for your interest in contributing to `fapethedev/workflows`! We welcome contributions to make our reusable GitHub Actions workflows more robust, secure, and developer-friendly.

Please take a moment to review this guide before submitting issues or pull requests.

---

## Code of Conduct

We are committed to providing a welcoming, inclusive, and harassment-free environment for all contributors. Please be respectful, constructive, and collaborative in all interactions across issues, discussions, and pull requests.

---

## How Can You Contribute?

You can contribute in many ways:
- **Report Bugs**: Report unexpected failures, syntax problems, or edge-case breakages in existing reusable workflows.
- **Propose New Workflows**: Suggest workflows for additional stacks (e.g., Rust, Java/Gradle, Terraform, Kubernetes).
- **Improve Security**: Enhance hardening, permissions isolation, and secret handling.
- **Improve Documentation**: Provide better examples, troubleshooting guides, or input/output parameter tables.

---

## Development Workflow

### 1. Fork & Branch
1. Fork the repository on GitHub.
2. Clone your fork locally:
   ```bash
   git clone https://github.com/<your-username>/workflows.git
   cd workflows
   ```
3. Create a feature or bugfix branch using standard naming conventions:
   - `feat/<feature-name>` (e.g., `feat/ci-rust-workflow`)
   - `fix/<bug-name>` (e.g., `fix/docker-buildx-cache-key`)
   - `docs/<doc-update>` (e.g., `docs/readme-security-examples`)
   - `ci/<ci-tweak>` (e.g., `ci/actionlint-rules`)

### 2. Standards for Reusable Workflows

Every workflow in `.github/workflows/` intended for reuse **must** adhere to the following architectural requirements:

#### A. Explicit `workflow_call` Interface & Casing Conventions
Declare all inputs, secrets, and outputs clearly with comprehensive descriptions. Respect standard casing conventions:
- **`inputs`**: Use lowercase `snake_case` (e.g., `gitlab_project_path`, `node_version`).
- **`secrets`**: Use uppercase `UPPER_SNAKE_CASE` (e.g., `GITLAB_TOKEN`, `NPM_TOKEN`), as secret names are case-sensitive across GitHub Actions.

```yaml
name: Node CI

on:
  workflow_call:
    inputs:
      node_version:
        description: 'Node.js version to test against (e.g., 20.x or matrix json)'
        required: false
        type: string
        default: '20.x'
      run_tests:
        description: 'Whether to execute the test suite'
        required: false
        type: boolean
        default: true
    secrets:
      NPM_TOKEN:
        description: 'NPM registry access token for private packages'
        required: false
    outputs:
      test_result:
        description: 'Status of the test execution'
        value: ${{ jobs.test.outputs.status }}
```

#### B. Fallback Strategy with Repository Variables (`vars`)
When designing reusable workflows, support both inline YAML inputs and GitHub Repository/Organization Variables (`vars`). This enables consumer repositories to keep their YAML minimal without hardcoding configuration in code:

```yaml
env:
  GITLAB_TOKEN: ${{ secrets.GITLAB_TOKEN }}
  GITLAB_PROJECT_PATH: ${{ inputs.gitlab_project_path || vars.GITLAB_PROJECT_PATH || github.repository }}
```

#### C. Isolated Concurrency Scoping
Always namespace concurrency groups with `${{ github.repository }}` so concurrent runs across different caller repositories do not cancel or queue each other:

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.repository }}-${{ github.ref }}
  cancel-in-progress: false
```

#### D. Principle of Least Privilege (`permissions`)
Every job must explicitly state its minimal required permissions:

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    permissions:
      contents: read
```

Never leave permissions implicit or set `permissions: write-all`.

#### C. Prevent Shell Script Injections
Never inject `${{ ... }}` context expressions directly inside `run:` blocks. Always pass untrusted inputs or context variables through `env`:

```yaml
# ❌ INSECURE: Vulnerable to script injection
- name: Echo issue title
  run: echo "${{ github.event.issue.title }}"

# ✅ SECURE: Passed via environment variable
- name: Echo issue title
  env:
    ISSUE_TITLE: ${{ github.event.issue.title }}
  run: echo "$ISSUE_TITLE"
```

#### D. Safe Third-Party Action Pinning
Pin all non-first-party GitHub Actions to a full 40-character commit SHA with a comment indicating the semantic version:

```yaml
- uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
```

---

## Local Validation & Testing

Before opening a pull request, verify your workflows locally:

### 1. Lint with `actionlint`
Install [`actionlint`](https://github.com/rhysd/actionlint) and run:

```bash
actionlint
```

Fix any syntax errors, untyped property access, or invalid expressions flagged by the tool.

### 2. Format YAML
Ensure YAML formatting is clean, properly indented (2 spaces), and has no trailing whitespace.

### 3. Local Dry-Run with `act` (Optional)
You can test workflows locally using [nektos/act](https://github.com/nektos/act):

```bash
act workflow_call -W .github/workflows/ci-node.yml
```

---

## Commit Message Guidelines

We follow the [Conventional Commits](https://www.conventionalcommits.org/) standard. Commit messages must be structured as follows:

```text
<type>(<scope>): <short description>

[optional body]

[optional footer(s)]
```

### Allowed Types
- **`feat`**: Adds a new reusable workflow or introduces a new feature to an existing workflow.
- **`fix`**: Bug fixes or correcting broken steps in workflows.
- **`docs`**: Documentation changes (README, SECURITY, CONTRIBUTING, comments).
- **`ci`**: Changes to repo-internal CI workflows (e.g. actionlint configuration).
- **`refactor`**: Code restructuring without changing workflow behavior.
- **`perf`**: Performance optimizations (e.g., improved caching strategies).
- **`chore`**: Maintenance, updating dependencies, or repo configuration.

### Examples
- `feat(ci-python): add support for uv package manager`
- `fix(docker-publish): resolve cache push failure on tag builds`
- `docs(readme): add security hardening section with SHA pinning examples`

---

## Pull Request Process

1. Ensure your branch is rebased onto the latest `master`/`main` branch.
2. Verify that `actionlint` passes without warnings.
3. If your changes alter the public interface of a workflow (inputs, outputs, secrets, behavior), update the documentation in [README.md](README.md) and record the change in [CHANGELOG.md](CHANGELOG.md).
4. Fill out the Pull Request template completely, detailing:
   - Purpose of the change
   - Motivation and context
   - How the workflow was verified/tested
5. Maintainers will review your PR, suggest any necessary adjustments, and merge once all checks pass.

---

## Releases & Versioning

This project adheres to [Semantic Versioning 2.0.0](https://semver.org/):
- **MAJOR** (`v2.0.0`): Breaking changes to workflow inputs, removed inputs/outputs, or increased required permissions.
- **MINOR** (`v1.1.0`): Backwards-compatible new features, new reusable workflows, or new optional inputs.
- **PATCH** (`v1.0.1`): Backwards-compatible bug fixes and security patches.

Whenever a new minor or patch release is cut (e.g. `v1.2.0`), the corresponding major floating tag (e.g. `v1`) is automatically updated to point to the latest stable release.
