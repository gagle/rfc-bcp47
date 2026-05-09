# Commit Conventions

This repo uses [Conventional Commits](https://www.conventionalcommits.org/).

Format: `type(scope): description`

- **type**: `feat` | `fix` | `refactor` | `chore` | `docs` | `test` | `style` | `ci` | `perf` | `build`
- **scope**: optional; omit if change isn't naturally scoped
- **description**: imperative mood, lowercase, no trailing period

Examples (from this repo's history):

- `fix(deps): refresh devDependencies, patches postcss CVE-2026-41305`
- `chore: release v1.5.0`
- `docs(readme): add Built with AI badge`

CHANGELOG.md is manual — entries are categorized under `### Features` /
`### Bug Fixes` / `### Documentation` / `### Miscellaneous Chores` based
on commit type. The release flow is tag-triggered (`release.yml`); no
automated tooling parses commit messages, so this convention is for
**human readability and CHANGELOG curation**, not runtime enforcement.
