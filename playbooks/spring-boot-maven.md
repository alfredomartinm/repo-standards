# Playbook: Spring Boot + Maven

This playbook covers how to apply `repo-standards` to a Spring Boot project built with Maven.

---

## Repository Tier Checklist

Use the checklist that matches the maturity of your repo.

### Lab / Experimental Repo

A repo used for learning, prototyping, or early-stage work.

- [ ] `README.md` exists with a brief description and local setup instructions
- [ ] `.gitignore` includes standard Java/Maven and IDE entries
- [ ] CI workflow runs `mvn -B verify` on push and pull_request
- [ ] Dependabot is configured for Maven (`pom.xml`) and GitHub Actions
- [ ] Branch protection is enabled on `main` (at minimum: require PRs)

### Portfolio Repo

A repo intended to showcase production-quality work or act as a reference implementation.

Everything in the lab checklist, plus:

- [ ] `POLICY.md` (or a link to this standards repo) is present
- [ ] `.github/PULL_REQUEST_TEMPLATE.md` matches the template in [`POLICY.md`](../POLICY.md)
- [ ] `CODEOWNERS` file defines at least one owner
- [ ] Branch protection requires at least one passing CI check before merge
- [ ] Housekeeping workflow is scheduled (monthly)
- [ ] Dependabot auto-merge is enabled for patch/minor updates (optional but recommended)
- [ ] Release tags follow `vMAJOR.MINOR.PATCH` convention
- [ ] `CHANGELOG.md` or GitHub Releases are used to document changes

---

## What the Housekeeping Workflow Checks

When `reusable-housekeeping-springboot-maven.yml` runs, it inspects the following:

| Check | Safe to auto-fix? | Notes |
|---|---|---|
| `.editorconfig` present and up to date | Yes | Applies standard Java/Maven settings |
| `PULL_REQUEST_TEMPLATE.md` matches standard | Yes | Overwrites only if outdated |
| CI workflow references pinned to current standard tag | Yes | Updates `@v1` references if a new major is available |
| `POLICY.md` present | Yes | Copies standard policy if missing |
| Java version in `pom.xml` matches global default (21) | **No** – suggested only | Logged in report; not auto-applied |
| Outdated Maven wrapper version | **No** – suggested only | Logged in report |
| Deprecated Spring Boot properties in `application.properties` | **No** – suggested only | Logged in report |

In `dry-run` mode, all findings are written to `HOUSEKEEPING_REPORT.md` and uploaded as a workflow artifact.
In `apply` mode, safe fixes are committed to a new branch and a PR is opened against `target_branch`.

---

## How to Interpret Dependency PRs

Dependabot opens PRs for outdated dependencies. Here's how to handle them:

### Patch updates (e.g., `3.2.0` → `3.2.1`)

- **Risk:** Very low.
- **Action:** Let auto-merge handle it if enabled. If CI passes, merge.

### Minor updates (e.g., `3.2.0` → `3.3.0`)

- **Risk:** Low to medium. Minor versions may add deprecations.
- **Action:** Auto-merge if CI passes. Review the changelog briefly if CI fails.

### Major updates (e.g., `3.x` → `4.x`)

- **Risk:** Medium to high. May include breaking API changes.
- **Action:** **Do not auto-merge.** Read the migration guide. The PR description should include a risk note. Test locally before merging.

### Risk notes in PR descriptions

The `reusable-dependabot-automerge.yml` workflow adds a comment to major-version PRs explaining:
- The version range being updated.
- A reminder to review the upstream changelog or migration guide.
- Instructions on how to test before merging.

If you see a PR labeled `breaking-change`, treat it as a major update regardless of the semver increment.

---

## Recommended Dependabot Configuration

Add `.github/dependabot.yml` to your repo:

```yaml
version: 2
updates:
  - package-ecosystem: maven
    directory: /
    schedule:
      interval: weekly
    open-pull-requests-limit: 10
    groups:
      spring-boot:
        patterns:
          - "org.springframework.boot:*"

  - package-ecosystem: github-actions
    directory: /
    schedule:
      interval: weekly
```

This groups Spring Boot dependency updates into a single PR, reducing noise.
