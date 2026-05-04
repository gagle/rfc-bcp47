---
name: npm-trust-setup
description: >
  Guided wizard to enable or extend npm OIDC Trusted Publishing across the
  packages in the current repo. Resolves the right `npm-trust` invocation,
  detects the workspace shape, shows current trust state, filters to packages
  still needing setup, walks the user through any required manual steps
  (`npm login`, web 2FA, `npm publish`), runs the configuration, and verifies
  the result.
---

# Setup npm-trust

Interactive wizard for `npm-trust`. The CLI handles detection, filtering,
and per-package configuration. This skill orchestrates the end-to-end flow:
gather inputs → confirm → handle manual steps → execute → verify.

## When to use

- First-time OIDC trust setup for a repo's published packages.
- Incremental setup after publishing one or more new packages.
- Auditing — checking which packages are or aren't trust-configured.

## Pre-flight — resolve the CLI invocation

Different hosts reach `npm-trust` differently (source checkout, devDep,
global install, npx fetch). Decide which to use **once**, then use the same
invocation in every step below. Try in this order and stop at the first match:

1. **Source checkout.** If `./bin/npm-trust.js` exists AND
   `node -p "require('./package.json').name"` prints `npm-trust`, use:
   ```
   <CLI> = node ./bin/npm-trust.js
   ```
2. **Local devDependency.** If `./node_modules/.bin/npm-trust` is
   executable:
   ```
   <CLI> = ./node_modules/.bin/npm-trust
   ```
3. **Global install.** If `command -v npm-trust` succeeds:
   ```
   <CLI> = npm-trust
   ```
4. **Registry fetch (last resort).** Otherwise fall back to:
   ```
   <CLI> = npx -y npm-trust@latest
   ```

If your shell's `npx` form rejects the package (some npm 11 setups require
`npm exec --` instead), substitute `npm exec -- npm-trust@latest` for
the npx fallback.

Use the chosen invocation in **every** subsequent step. Below, `<CLI>` is the
placeholder — replace mentally.

### Verify the resolved version supports the flags this skill uses

```bash
<CLI> --help 2>/dev/null | grep -q -- "--auto" \
  && echo "ok" \
  || echo "TOO_OLD"
```

If the result is `TOO_OLD`, the resolved binary predates `--auto` (added in
v0.2.0). **Stop** and ask the user to upgrade with
`npm i -g npm-trust@latest` (or remove the cached version that resolved).

## Phase 1 — Discover (one call when --doctor is supported)

If the resolved CLI supports `--doctor` (added in v0.4.0), the entire Phase 1
sequence below collapses to a single call:

```bash
<CLI> --doctor --json
```

The JSON has the shape `{ schemaVersion: 1, runtime, auth, workspace, repo,
workflows, packages, issues, summary }`. Branch on:

- `summary.fail > 0` — STOP. The environment can't run the trust setup.
- `issues[]` — surface every `code` to the user, with the included `remedy`
  text where present. Notable codes: `AUTH_NOT_LOGGED_IN`,
  `WORKSPACE_NOT_DETECTED`, `WORKFLOWS_AMBIGUOUS`, `PACKAGE_NOT_PUBLISHED`,
  `PACKAGE_TRUST_DISCREPANCY`.
- `workspace.packages[]` — the detected package list.
- `repo.inferredSlug` — `<owner/repo>` for the configure call.
- `workflows[]` — candidate workflow files; if length === 1, suggest it.

If `--doctor` is not present in `<CLI> --help`, fall back to the per-step
discovery below. The skill is fully backward-compatible with v0.2.0 and
v0.3.0 CLIs.

### 1. Detect the workspace shape

```bash
<CLI> --auto --dry-run --repo placeholder/x --workflow placeholder.yml 2>&1 | head -10
```

Expected first stdout line:

- `Detected pnpm workspace — found N packages` for repos with `pnpm-workspace.yaml`.
- `Detected npm/yarn workspace — found N packages` for `package.json#workspaces`.
- `Detected single package — found 1 packages` for repos with a single root `package.json`.

If the output is `Error: --auto could not detect…`, call `AskUserQuestion`
(loading via `ToolSearch query="select:AskUserQuestion"` if needed) with:

- `header`: `"Source flag"`
- `question`: `"--auto couldn't detect the workspace. How should packages be discovered?"`
- options:
  1. `Use --scope` — Discover via npm registry under a scope (e.g. `--scope @myorg`)
  2. `Use --packages` — Explicit list of package names
  3. `Abort` — Stop the wizard

Use the chosen source flag in every subsequent step. For `Use --scope` and
`Use --packages`, follow up free-form to capture the scope or names.

### 2. Resolve the GitHub repo

```bash
git remote get-url origin
```

Parse `owner/repo` from the URL, then call `AskUserQuestion` to confirm:

- `header`: `"GitHub repo"`
- `question`: `"Use {inferred owner/repo} as the GitHub repo for OIDC trust?"`
- options:
  1. `Confirm` — Use the inferred slug
  2. `Override` — Specify a different `owner/repo` (free-form follow-up)
  3. `Abort` — Stop the wizard

### 3. Resolve the publish workflow

List candidate workflow files:

```bash
ls .github/workflows/*.yml .github/workflows/*.yaml 2>/dev/null
```

If exactly one file exists, use it without prompting. Otherwise call
`AskUserQuestion`:

- `header`: `"Workflow"`
- `question`: `"Which workflow performs the npm publish?"`
- options: one entry per candidate file (label is the basename, e.g.
  `release.yml`). Cap at 4 options. If more than 4 candidates exist,
  show the 3 most likely (any file containing `release`, `publish`, or
  `npm` in the name) plus rely on the auto-`Other` choice for the rest.

### 4. Show current trust state

```bash
<CLI> --auto --list
```

Per-package output: each line shows the package name and either its existing
trust configuration or `(no trust configured)`.

> **Note on cross-checking.** `npm trust list` (which the CLI calls under the
> hood) sometimes reports `(no trust configured)` even when OIDC publishing
> demonstrably works — this happens when Trusted Publishing was set up via
> npm's web UI rather than the CLI, or when the CLI's auth state is stale.
> Recent versions of `npm-trust` cross-check via `npm view <pkg>
> dist.attestations` to flag this; if you see a `(provenance present)` marker
> in the output, treat the package as effectively configured even if the
> trust line is empty.

### 5. Identify what still needs work

```bash
<CLI> --auto --only-new --list
```

The filter calls `npm trust list`, `npm view <pkg>`, and (in v0.3.0+)
`npm view <pkg> dist.attestations` per package. It keeps packages that lack
trust **and** lack provenance, or aren't yet published. The result is the
precise working set for this run.

### 6. Confirm with user

Render the summary as text (so it stays visible in the chat):

```
Detected: <source>, <N> packages
Repo:     <owner/repo>
Workflow: <file>

Already configured: <K>
Needs work:         <M>

Plan: configure OIDC trust for <M> packages.
```

Then call `AskUserQuestion` to gate Phase 2:

- `header`: `"Setup plan"`
- `question`: `"Proceed with the plan above?"`
- options:
  1. `Proceed` — Run Phase 2 (auth gate, dry-run, configure, verify)
  2. `Abort` — Stop the wizard, no state changes

## Phase 2 — Execute

### 7. Auth gate (hard)

Run:

```bash
npm whoami
```

If it **fails or prints nothing**, **STOP**. Tell the user:

> You're not logged in to npm. Run `npm login` in another terminal, then
> re-run this skill from Step 1.

Do not proceed to the configure step until `npm whoami` succeeds. The
configure step uses web 2FA which won't work without an authenticated
session.

Once logged in, surface the pre-auth notice:

> The first `npm trust github` call will open a browser for npm
> authentication. On the npm site, tick **"skip 2FA for the next 5 minutes"**
> so the remaining packages finish without further prompts.

### 8. Pre-flight dry-run

Before burning a 2FA round-trip on a typo, validate the args:

```bash
<CLI> --auto --only-new --dry-run --repo <owner/repo> --workflow <file>
```

If this errors (bad repo regex, invalid workflow extension, source detection
failure), **stop and surface the error** — do not proceed to the actual
configure call.

### 9. Configure trust

```bash
<CLI> --auto --only-new --repo <owner/repo> --workflow <file>
```

The CLI reports per-package status and a final summary
(`Done: X configured, Y already set, Z failed`).

### 10. Handle unpublished packages

If the summary lists failed packages with the suffix `not published yet`:

> The following packages need to be published before OIDC trust can be
> configured for them:
>
> - @scope/new-pkg-a
> - @scope/new-pkg-b
>
> Publish them via your normal release process, then re-run this skill — only
> the now-published packages will be picked up.

Do not retry automatically; publishing is a deliberate user action.

### 11. Verify

```bash
<CLI> --auto --list
```

All previously-untrusted-but-published packages should now show trust
information (the workflow file path) instead of `(no trust configured)`.

### 12. Report

Print a final summary:

```
Setup complete.
  Configured this run:  <X>
  Already set:          <Y>
  Pending publish:      <Z>
```

If `Z > 0`, remind the user that those packages still need publishing.

## Notes

- Scope-based mode (`--scope @myorg`) is an alternative to `--auto`. Use it
  when the user explicitly wants to discover via the npm registry rather than
  the workspace filesystem — useful for cross-cutting audits of an entire org.
- `--only-new` is filesystem-scoped: it filters whichever package list was
  produced by the source flag (`--auto`, `--scope`, or `--packages`). Combine
  freely.
- The wizard never runs destructive commands; everything it executes is
  read-only or `npm trust github` (which is idempotent — re-running on an
  already-configured package returns "already configured").
- `--doctor` is the fast path for v0.4.0+ CLIs (see Phase 1 opener above).

## Recommended conventions

These are conventions the maintainer recommends for repos that adopt
`npm-trust`. They are not enforced by the CLI — adopt them for
consistency across projects.

### Use `AskUserQuestion` for every human decision

When this skill (or any skill that builds on it) needs a decision from
the user — proceed/abort, pick a workflow, override an inferred value —
the agent **must** call the `AskUserQuestion` tool. Do not write a free-
text question and stop. Free-text prompts are too easy to miss against
the surrounding agent output; the structured selector is unmissable.

### npm script naming: `npm-trust:setup`, not `setup:npm-trust`

Recommend that consumers expose the wizard under a `npm-trust:setup`
script (namespace before verb), so scripts grouped by tool sort together
and match the prevailing pnpm/monorepo convention:

```json
{
  "scripts": {
    "npm-trust:setup": "npm-trust --auto --repo <owner/repo> --workflow <file>"
  }
}
```

The older `setup:npm-trust` ordering is discouraged; rename when you
encounter it.
