# rfc-bcp47 — Claude Code context

## Project

`rfc-bcp47` is a **zero-dependency** TypeScript/JavaScript library that parses,
normalizes, and matches BCP 47 language tags (RFC 5646 + RFC 4647 lookup /
filtering). It ships as a dual ESM + CJS package and exposes a single public
entry point at `src/index.ts`.

The package is published on the npm registry at
`https://www.npmjs.com/package/rfc-bcp47` and is OIDC-trust-bootstrapped: every
release lands with a SLSA provenance attestation.

## Stack

- TypeScript 6.x strict ESM — `"type": "module"`, `target: ES2022`,
  `module: ESNext`, `moduleResolution: bundler`.
- Node.js **>= 24** (see `package.json#engines`; pinned via `.nvmrc`).
- **tsdown** for the build (single-pass bundler producing ESM + CJS + types +
  source maps in one go).
- Vitest + `@vitest/coverage-v8` for tests. No enforced coverage threshold.
- husky for git hooks. **No eslint** — `lint` is `tsc --noEmit`.
- pnpm 10.15.0 (pinned via `packageManager`).

## Layout

```
src/
  index.ts                       # PUBLIC API — re-exports only, no side effects
  language-subtag-registry.ts    # IANA registry data + lookup helpers
  types.ts                       # public types (BCP47Tag, ParseResult, etc.)
  utilities.ts                   # internal helpers
  operators/                     # parse / normalize / match / filter / lookup
  *.spec.ts                      # unit tests (co-located with source)
  integration.spec.ts            # cross-operator integration tests
examples/                        # *.ts snippets that import from 'rfc-bcp47'
                                 # — type-checked via tsconfig.examples.json
github-pages/                    # hand-curated docs site (deployed via pages.yml)
docs/                            # extra documentation
.github/workflows/
  ci.yml                         # lint + test on push / PR
  release.yml                    # tag-triggered publish (with --provenance)
  pages.yml                      # GitHub Pages deploy
.claude/                         # rules, skills (see below)
```

## Commands

```bash
pnpm run lint         # tsc --noEmit -p tsconfig.examples.json (typechecks src + examples)
pnpm run build        # tsdown → dist/index.{mjs,cjs,d.mts,d.cts}
pnpm test             # vitest run
pnpm run test:watch   # vitest in watch mode
pnpm run test:coverage # vitest run --coverage (no threshold enforced)
pnpm run dev:pages    # build + copy ESM bundle to github-pages/
pnpm run npm-trust:setup  # run the OIDC trust setup wizard (npm-trust@0.6.0)
```

## Conventions

Project-local rules live in `.claude/rules/`:

- `typescript.md` — TS conventions (`Array<T>`, `readonly`, no `any`, naming,
  no `.js` extensions in source imports — tsdown handles bundling).
- `testing.md` — Vitest patterns (18 enforced rules around describe/it
  structure, assertions, mocks). No enforced coverage threshold.
- `review-criteria.md` — five-axis review checklist.

User-global conventions in `~/.claude/rules/` (TypeScript / Angular / SCSS /
a11y) also apply where relevant — but this is a pure-TS library, so the
Angular / SCSS / a11y rules are not used.

## Skills

`.claude/skills/`:

- `commit` — git workflow (stage by name, conventional commits, squash, push).
- `verify` — thin wrapper around `/solo-npm:verify` (lint → build → test). Run before marking work complete.
- `release` — thin wrapper around `/solo-npm:release` (version bump + CHANGELOG + tag → CI publishes via OIDC + provenance).
- `review` — five-axis principal review.
- `testing` — Vitest test templates with BCP 47 examples.

The release/verify wrappers compose with the [`gagle/solo-npm`](https://github.com/gagle/solo-npm)
marketplace plugin. Other lifecycle skills are invoked directly without
a wrapper:

- `/solo-npm:trust` — OIDC trust setup wizard (rare; for incremental config).
- `/solo-npm:status` — read-only portfolio dashboard.
- `/solo-npm:audit` — security audit with risk triage.
- `/solo-npm:deps` — dep upgrade orchestrator with verify gates.
- `/solo-npm:init` — fresh-repo bootstrap (umbrella; not needed for this repo).

## Release flow

Tag-triggered: push a `v*` tag → `.github/workflows/release.yml` runs
lint / test / build / `pnpm publish --no-git-checks --provenance --access
public` with `id-token: write` permission. The OIDC publisher identity is
trusted by the registry, so `--provenance` produces a signed SLSA attestation
on every release.

The previous release flow (`googleapis/release-please-action`) was retired in
favour of manual versioning + CHANGELOG. Use the `release` skill to drive
the manual steps.

## Important guardrails

- **No `.js` extensions in TS imports.** `moduleResolution: bundler` lets
  tsdown resolve `import { x } from './foo'` correctly. Adding `.js` is wrong
  for this repo.
- **`src/index.ts` is the public API surface.** Only the symbols re-exported
  there are stable. Internal helpers in `language-subtag-registry.ts`,
  `utilities.ts`, and `operators/` stay unexported.
- **Zero runtime dependencies.** `package.json#dependencies` is intentionally
  empty. devDependencies are fine; runtime deps require explicit justification.
- **Don't break the dual ESM/CJS surface.** `package.json#exports` is the
  contract. tsdown emits both; changes that drop one would break consumers.
- **Node engine is pinned to `>=24`** via `package.json#engines` and `.nvmrc`.
  All workflows read the version from `.nvmrc` (`node-version-file: .nvmrc`),
  so a bump is a one-line edit. Source uses ES2024 features (`toSorted`,
  `at(-1)`, etc.) that depend on the Node 24 baseline.
