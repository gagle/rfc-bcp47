---
name: verify
description: >
  Run all quality gates: lint, build, unit tests. Use after any code change to
  ensure nothing is broken before marking work complete.
---

# Verify

Run each step sequentially. On failure, stop and report which step failed with
the error output. Do not proceed to the next step until the current one passes.

## Steps

### 1. Lint (= typecheck via tsc --noEmit, covers src + examples)

```bash
pnpm run lint
```

`lint` runs `tsc --noEmit -p tsconfig.examples.json`, which type-checks both
`src/` and `examples/` in one pass. The `tsconfig.examples.json` adds a path
mapping (`rfc-bcp47` → `./src/index.ts`) so the example files — which import
from the package by name — resolve to the local source. This catches API
breakage in either the public exports or the example snippets.

There is no separate `typecheck` script.

### 2. Build

```bash
pnpm run build
```

Runs `tsdown` and emits `dist/index.{mjs,cjs,d.mts,d.cts}`.

### 3. Unit tests

```bash
pnpm test
```

Runs the vitest suite under `src/`. Coverage thresholds are not enforced;
inspect coverage manually with `pnpm run test:coverage` when relevant.

## On failure

1. Read the error output and diagnose the root cause.
2. Fix the issue.
3. Re-run from the failed step (not from the beginning — earlier steps already passed).
4. Repeat until all steps pass.

## Report

After all steps pass, print a summary:

```
Verification complete:
  ✓ Lint
  ✓ Build
  ✓ Unit tests
```
