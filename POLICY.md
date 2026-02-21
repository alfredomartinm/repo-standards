# Repository Standards Policy

This document describes the principles and rules that govern repositories adopting `alfredomartinm/repo-standards`.

---

## Purpose and Principles

- **Consistency** – All maintained repos follow the same structure, tooling, and process.
- **Safety first** – Automated changes touch only low-risk areas (docs, meta, workflows, templates). Code refactors are suggested in reports, not applied automatically.
- **Minimal friction** – Housekeeping is lightweight. The goal is a tidy repo, not busy-work.
- **Transparency** – Every automated change opens a PR with a clear description, risk note, and checklist.

---

## Global Defaults

| Setting | Value |
|---|---|
| Java version | 21 |
| Dependency update cadence | Weekly (via Dependabot) |
| Housekeeping cadence | Monthly (first of the month) once a repo is stable |
| Default branch | `main` |
| Merge strategy | Squash merge |

---

## Housekeeping Philosophy

The housekeeping workflow is intentionally lightweight.

**Safe to auto-change (apply mode):**
- Outdated or missing `.editorconfig` settings
- Stale CI workflow versions referenced in caller workflows
- Missing or outdated standard files: `POLICY.md`, `.github/PULL_REQUEST_TEMPLATE.md`, `CODEOWNERS`
- Workflow template updates (non-breaking)

**Suggested only (never auto-applied):**
- Code refactors (Java source, configuration beans, etc.)
- Changes to `pom.xml` beyond dependency versions managed by Dependabot
- Removal of deprecated APIs unless explicitly opted in

If the workflow detects suggested changes but cannot apply them safely, it uploads a `HOUSEKEEPING_REPORT.md` artifact with findings for manual review.

---

## Dependency Update Policy

Dependency updates are managed by [Dependabot](https://docs.github.com/en/code-security/dependabot).

### Update cadence

- **Weekly** for all ecosystems (Maven, GitHub Actions).

### Merge strategy by update type

| Update type | Strategy | Notes |
|---|---|---|
| Patch | Auto-merge (when checks pass) | Safe; no API changes |
| Minor | Auto-merge (when checks pass) | Generally safe; review changelog if CI fails |
| Major | Manual review required | PR includes a risk explanation; do not merge until reviewed |

### Enabling auto-merge

Auto-merge for patch/minor is **opt-in per repo**. See [`README.md`](README.md#enabling-auto-merge-for-dependabot-prs) for setup instructions.

Auto-merge is **currently not enabled** in this (`repo-standards`) repository.

### Major version updates

Major version bumps may include breaking changes. The Dependabot PR should include:
- A summary of what changed (pulled from the release notes or changelog).
- A risk explanation (e.g., "This upgrades Spring Boot from 3.x to 4.x; review migration guide before merging").
- Manual confirmation before merging.

---

## PR Conventions

### Labels

| Label | Meaning |
|---|---|
| `dependencies` | Dependency update PR (applied automatically by Dependabot / automerge workflow) |
| `automerge` | PR is eligible for auto-merge |
| `housekeeping` | Automated housekeeping PR |
| `breaking-change` | PR contains a breaking or major version change |
| `documentation` | Documentation-only change |

### PR Title Format

Follow [Conventional Commits](https://www.conventionalcommits.org/) style:

```
<type>(<scope>): <short description>
```

Examples:
- `chore(deps): bump spring-boot-starter-web from 3.2.0 to 3.2.1`
- `chore(housekeeping): update CI workflow template to v1.2`
- `feat(api): add health check endpoint`

### PR Body Template

All PRs (automated and manual) should include the following sections:

```markdown
## Summary
<!-- What does this PR do? -->

## Risk
<!-- Low / Medium / High. Explain why. -->
<!-- For dependency PRs: mention if this is a major version bump. -->

## Checklist
- [ ] Tests pass locally
- [ ] Relevant documentation updated
- [ ] No unintended side effects

## How to Test
<!-- Steps to verify the change works as expected. -->
```

A `.github/PULL_REQUEST_TEMPLATE.md` following this format is maintained as part of the housekeeping workflow.
