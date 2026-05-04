---
name: verify
description: Verify rfc-bcp47 — wraps /solo-npm:verify with this repo's lint+test+build commands.
---

# Verify (rfc-bcp47)

Composes /solo-npm:verify with this repo's specifics.

## Repo context

- Lint: `pnpm lint`
- Test: `pnpm test`
- Build: `pnpm build`

## Workflow

Invoke `/solo-npm:verify` for the opinionated baseline. Run the three
commands above sequentially; halt on first failure; surface full output.
