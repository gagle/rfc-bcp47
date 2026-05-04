---
name: release
description: Release rfc-bcp47 — wraps /solo-npm:release for this single-package repo.
---

# Release (rfc-bcp47)

Composes /solo-npm:release with this repo's specifics.

## Repo context

- Workspace: single package at repo root
- Repo slug: `gagle/rfc-bcp47`
- Workflow: `release.yml`
- Verification: `/verify` runs `pnpm lint && pnpm test && pnpm build`

## Workflow

Invoke `/solo-npm:release` for the opinionated three-phase baseline.

## Deviations from the baseline

(none)
