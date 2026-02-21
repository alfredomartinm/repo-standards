# repo-standards

A public, versioned collection of gold-standard templates, policies, and reusable GitHub Actions workflows for repository maintenance and presentation.

---

## Purpose

`alfredomartinm/repo-standards` provides:

- **Documented standards** for how repositories should be structured and maintained.
- **Reusable GitHub Actions workflows** that adopting repositories can call directly by tag.
- **An optional auto-merge mechanism** for Dependabot PRs (documented below; not enabled in this repo).

---

## Reusable Workflows

All reusable workflows live under [`.github/workflows/`](.github/workflows/).

| Workflow file | Description |
|---|---|
| `reusable-ci-maven.yml` | CI pipeline: Java 21, Maven cache, `mvn -B verify` |
| `reusable-housekeeping-springboot-maven.yml` | Lightweight housekeeping checks and optional updates |
| `reusable-dependabot-automerge.yml` | Auto-merge Dependabot patch/minor PRs when checks pass |

### Referencing workflows by tag

Adopting repos should always pin to a release tag (e.g., `v1`) rather than a branch:

```yaml
jobs:
  ci:
    uses: alfredomartinm/repo-standards/.github/workflows/reusable-ci-maven.yml@v1
```

When a new version is released, bump the tag in your caller workflow to opt in to the update.

---

## Required Permissions and Secrets

### `reusable-ci-maven.yml`

No additional secrets required. The default `GITHUB_TOKEN` is sufficient.

### `reusable-housekeeping-springboot-maven.yml`

| Input | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `dry-run` | `dry-run` generates a report; `apply` opens a PR |
| `target_branch` | string | `main` | Branch to target when opening a housekeeping PR |

Required permissions in the **caller** workflow:

```yaml
permissions:
  contents: write
  pull-requests: write
```

A `GITHUB_TOKEN` with those permissions (or a PAT stored as a secret) must be available as `secrets.GITHUB_TOKEN` or the default token.

### `reusable-dependabot-automerge.yml`

Required permissions in the **caller** workflow:

```yaml
permissions:
  contents: write
  pull-requests: write
```

The target repository **must have auto-merge enabled** in its settings before this workflow can set a PR to auto-merge.

---

## Rolling Out to a New Repo

1. Ensure the target repo has branch protection and required status checks configured.
2. Add a caller workflow in `.github/workflows/ci.yml`:

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:

jobs:
  ci:
    uses: alfredomartinm/repo-standards/.github/workflows/reusable-ci-maven.yml@v1
    permissions:
      contents: read
```

3. (Optional) Add a housekeeping workflow `.github/workflows/housekeeping.yml`:

```yaml
name: Housekeeping

on:
  schedule:
    - cron: '0 9 1 * *'  # monthly, first of the month
  workflow_dispatch:
    inputs:
      mode:
        description: 'dry-run or apply'
        default: 'dry-run'

jobs:
  housekeeping:
    uses: alfredomartinm/repo-standards/.github/workflows/reusable-housekeeping-springboot-maven.yml@v1
    with:
      mode: ${{ github.event.inputs.mode || 'dry-run' }}
      target_branch: main
    permissions:
      contents: write
      pull-requests: write
```

4. Commit and push. The workflows will run on the next trigger.

---

## Enabling Auto-Merge for Dependabot PRs

> **Note:** Auto-merge is **not** enabled in this repository. The workflow is provided for adopting repos.

### Prerequisites

1. Enable auto-merge for the repository in **Settings → General → Allow auto-merge**.
2. Configure branch protection rules requiring at least one status check to pass before merging.

### Setup

Add a caller workflow `.github/workflows/dependabot-automerge.yml`:

```yaml
name: Dependabot auto-merge

on:
  pull_request:

jobs:
  automerge:
    if: github.actor == 'dependabot[bot]'
    uses: alfredomartinm/repo-standards/.github/workflows/reusable-dependabot-automerge.yml@v1
    permissions:
      contents: write
      pull-requests: write
```

The workflow will:
- Apply the `dependencies` and `automerge` labels.
- Enable auto-merge (squash) only for **patch** and **minor** dependency updates.
- Leave **major** updates for manual review.

See [`POLICY.md`](POLICY.md) for the full dependency update policy.

---

## Self-Test Workflows

`repo-standards` ships two validation workflows that confirm the reusable workflows are syntactically correct and run end-to-end with their default configuration.

| Workflow file | What it validates |
|---|---|
| `validate-reusable-ci-maven.yml` | Calls `reusable-ci-maven.yml` with default inputs |
| `validate-reusable-housekeeping-springboot-maven.yml` | Calls `reusable-housekeeping-springboot-maven.yml` in `dry-run` mode (no commits or PRs are created) |

### Running a self-test

1. Navigate to **Actions** in this repository.
2. Select the desired validate workflow from the left-hand list.
3. Click **Run workflow** → **Run workflow**.

Both workflows are triggered manually (`workflow_dispatch`) and will complete successfully on `main` without side effects.

---

## Versioning

This repo follows [Semantic Versioning](https://semver.org/). Stable releases are tagged (e.g., `v1`, `v1.1.0`). Adopting repos should pin to the major version tag (e.g., `@v1`) so they automatically receive non-breaking updates.
