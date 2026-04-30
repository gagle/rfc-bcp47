---
name: release
description: >
  Version bump + changelog + tag + CI publish for rfc-bcp47. Local verification
  → bump version → commit → push → tag → CI publishes via OIDC + provenance.
  rfc-bcp47 is already trust-bootstrapped on npm; every release goes through CI.
---

# Release

Single-phase release: local verification → push commit → tag → CI publishes to npm.

OIDC Trusted Publishing is already wired for `rfc-bcp47` on the registry, and
the workflow at `.github/workflows/release.yml` runs `pnpm publish --provenance
--access public` on every `v*` tag push. Every release goes through CI; there
is no first-publish ceremony to perform.

## Phase 1 — Local

### 1. Guard

Verify clean working tree:

```bash
git status --porcelain
```

If non-empty, **stop** and tell the user to commit or stash first.

### 2. Fast verification

Run all local checks. Abort on first failure:

```bash
pnpm run lint && pnpm test && pnpm run build
```

### 3. Find latest version

```bash
git tag --list 'v*' --sort=-v:refname | head -1
```

### 4. Collect commits

```bash
git log <tag>..HEAD --format='%H %s'
```

Parse each as a conventional commit: `type(scope)?: subject`.

Ignore commits that don't match the conventional format (e.g., merge commits).

### 5. Determine version bump

| Condition                                                         | Bump                  |
| ----------------------------------------------------------------- | --------------------- |
| Any commit has `!` after type OR body contains `BREAKING CHANGE:` | **major**             |
| Any `feat` commit                                                 | **minor**             |
| Any `fix`, `perf`, or `revert` commit                             | **patch**             |
| None of the above                                                 | **no release** — stop |

The highest applicable bump wins.

### 6. Confirm with user

Print a summary:

```
Release: v{current} → v{next}

Breaking Changes:
  - subject (hash)

Features:
  - subject (hash)

Bug Fixes:
  - subject (hash)

N commits, M releasable
```

Ask the user to confirm before proceeding.

### 7. Generate changelog

Prepend a new section to `CHANGELOG.md`.

```markdown
## [X.Y.Z](https://github.com/gagle/rfc-bcp47/compare/vPREV...vX.Y.Z) (YYYY-MM-DD)

### Breaking Changes

- subject ([hash](https://github.com/gagle/rfc-bcp47/commit/hash))

### Features

- subject ([hash](https://github.com/gagle/rfc-bcp47/commit/hash))

### Bug Fixes

- subject ([hash](https://github.com/gagle/rfc-bcp47/commit/hash))

### Performance

- subject ([hash](https://github.com/gagle/rfc-bcp47/commit/hash))
```

Only include sections that have entries.

### 8. Bump version

Update the `"version"` field in `package.json` only (single-package repo).

### 9. Commit

```bash
git add CHANGELOG.md package.json
git commit -m "chore: release v{version}"
```

### 10. Push commit

```bash
git push
```

### 11. Final pre-publish verification

Re-run all gates against the bumped version. Abort on any failure:

```bash
pnpm run lint && pnpm test && pnpm run build
```

### 12. Tag

```bash
git tag v{version}
```

### 13. Push the tag

```bash
git push --tags
```

The push triggers `.github/workflows/release.yml`, which installs, lints,
tests, builds, and runs `pnpm publish --no-git-checks --provenance --access
public` with the GitHub OIDC token. The `id-token: write` permission lets npm
verify the publisher matches the trusted source.

### 14. Watch CI

```bash
gh run watch
```

If the publish step fails: keep the tag (it documents intent), fix the
publish-side issue, and re-run the workflow via `gh run rerun`. Do not bump
the version unless the failure was caused by tarball content that needs
another commit.

### 15. Verify

```bash
npm view rfc-bcp47@{version} version
npm view rfc-bcp47@{version} dist.attestations
```

The second call should show a SLSA provenance attestation tied to the release
commit. Notify the user:

> Released `v{version}` to npm. Tarball: https://www.npmjs.com/package/rfc-bcp47/v/{version}

## Failure recovery

If any local gate (steps 1–2 or 11) fails: fix and restart from step 1.

If the CI publish job fails after the tag is pushed: re-run the workflow once
the cause is fixed. Do not bump version unless the tarball needs new content.
