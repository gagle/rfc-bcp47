---
name: commit
description: >
  Composable git workflow: commit, squash, and/or push. Use /commit to stage and commit,
  /commit squash to squash branch commits, /commit push to force push,
  or /commit squash push for the full workflow.
---

# Commit

Composable git workflow with three independent actions. Parse `$ARGUMENTS` for: `squash`, `push`. If no arguments, default to **commit only**.

## Actions

### commit (default, always runs first when no args or when there are unstaged changes)

1. **Gather state** (parallel):
   - `git status` (never `-uall`)
   - `git diff` (staged + unstaged)
   - `git log --oneline -10` (message style reference)

2. **Stage files** — add all modified and untracked files by name. Never use `git add -A` or `git add .`.

3. **Analyze changes** — determine:
   - **type**: `feat` | `fix` | `refactor` | `chore` | `docs` | `test` | `style` | `ci` | `perf` | `build`
   - **scope**: must be one of the commitlint-enforced scopes: `eutils`, `pubmed-xml`, `pubmed`, `jats`, `pmc`, `id-converter`, `pubtator`, `mesh`, `cite`, `mcp`, `rate-limiter`, `xml`, `workspace`
   - **description**: concise summary of what changed and why

4. **Commit** — single-line message, no body, no `Co-Authored-By`:

   ```
   git commit -m "type(scope): description"
   ```

   Must pass commitlint validation (husky commit-msg hook enforces this).

5. **Verify** — `git log --oneline -1`

### squash (when `$ARGUMENTS` contains "squash")

1. **Determine base branch** — `git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed 's|refs/remotes/origin/||'` (fallback: `main`).

2. **Count commits** on the branch:

   ```
   git log --oneline origin/<base>..HEAD
   ```

   If only 1 commit, skip — nothing to squash.

3. **Read all commit messages** — synthesize a unified message that captures the full scope of work.

4. **Squash**:

   ```
   git reset --soft HEAD~N
   git commit -m "type(scope): unified description"
   ```

5. **Verify** — `git log --oneline -1`

### push (when `$ARGUMENTS` contains "push")

1. **Verify** — confirm there are commits ahead of remote.

2. **Push**:

   ```
   git push --force-with-lease
   ```

3. **Verify** — `git log --oneline -1` + confirm remote is up to date.

## Rules

- Never use `git add -A` or `git add .` — add files by name
- Single-line commit messages only — no body, no `Co-Authored-By`
- Always use `--force-with-lease` over `--force`
- Commit messages must be commitlint-compatible (conventional commits with enforced scope enum)
- Squash synthesizes a new message from all branch commits — not just the first one
- If `squash push` is given, run squash first, then push
