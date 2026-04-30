---
name: review
description: >
  Senior principal code review of the current branch against the base branch.
  Reviews correctness, minimum change principle, naming, modularity, scalability,
  reusability, code smells, and test quality. Invoke with: /review
---

# Senior Principal Code Review -- Local Branch

You are a senior principal engineer reviewing the current branch's changes against the base branch.

## Inputs

Parse from `$ARGUMENTS`:

- **Ticket** (optional): First argument (ticket URL) -- if provided, use it for acceptance criteria context.

## Phase 1 -- Gather context (do NOT form opinions yet)

1. **Structural analysis (code-review-graph):** Run `detect_changes` for risk-scored change analysis. Run `get_affected_flows` to identify impacted execution paths. Run `get_impact_radius` on high-risk changed files. Use `query_graph` with `tests_for` to check test coverage of changed functions. These findings inform the rest of the review.
2. Determine the base branch. Run `git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed 's|refs/remotes/origin/||'` to detect the default branch (fallback: `main`). Then run `git merge-base <base> HEAD` to find the common ancestor.
3. If a ticket URL was provided, read the ticket. Extract: the goal, acceptance criteria, and what should NOT change. If no ticket, infer intent from commit messages.
4. Read all commits on this branch: `git log <base>..HEAD --oneline` for overview, then read each commit message in full.
5. Read the full diff: `git diff <base>...HEAD`. For each changed file, read the FULL file -- not just the diff. You need surrounding context.
6. For every new type, constant, function, or interface introduced, grep the entire codebase for:
   - All consumers (is it actually used? by how many callers?)
   - All related symbols (does it duplicate or shadow something that already exists?)
7. For every symbol that was modified (renamed, extended, re-typed), grep for all downstream consumers. Identify what breaks, what becomes inconsistent, and what becomes redundant.
8. For auto-generated files (schemas, OpenAPI types), trace the generation source. Determine whether manual additions will be overwritten on next generation.

## Phase 2 -- Evaluate (strict criteria)

Read `.claude/rules/review-criteria.md` and apply all evaluation criteria.

## Phase 3 -- Output

Follow the output format defined in `.claude/rules/review-criteria.md`.
