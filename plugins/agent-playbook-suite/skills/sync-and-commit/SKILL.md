---
name: sync-and-commit
description: Verify the work is consistent, complete, and accurate; sync the docs tree; commit and push. Use at the end of a TDD phase or step (especially when called from ship-milestone), or whenever the operator wants to wrap up in-flight work. Reads CLAUDE.md for project conventions; runs `docs check` against the docs tree; never bypasses git hooks; never pushes to `main`.
---

# sync-and-commit

End-of-step wrap-up. Verify the work, sync the docs tree to
reality, and commit (and push, when on a feature branch with a
remote).

Optionally pairs with [`docs-cli`](https://github.com/ArtRichards/docs-cli)
**v1.2.0 (M7+)** for the `Lifecycle:` / `Role:` / `Updated:`
metadata convention used by `project-foundation`,
`create-milestones`, and `ship-milestone`. If the project does
not use docs-cli, the docs-side checks are skipped and only the
project's own verification commands run.

## Step 1 — Read project context

Read `CLAUDE.md` at the repo root for:

- The docs-tree location (if any) and which artifacts live in it.
- Build/test/quality commands.
- Commit message conventions.
- Branch conventions (especially: which branches are safe to
  push to vs. which require operator review).
- Any project-specific rules.

If CLAUDE.md is missing, fall back to inspecting recent
`git log --oneline` for commit style and `package.json` /
`pyproject.toml` / `Makefile` for commands.

## Step 2 — Technical verification

Quick sanity pass: read through the diff. Anything obviously
wrong or incomplete? Then walk the checklist.

### 2a. Code completeness

For the phase/step just completed, verify:

- All required functionality is implemented (check against the
  milestone spec).
- No placeholder code, `TODO`/`FIXME` comments, or
  "implement later" stubs left behind.
- No commented-out code that should be removed.
- All edge cases from the spec are handled.
- Error handling is in place where needed.
- No dead code or unused imports.

### 2b. Test completeness

- Unit tests for new functions/methods.
- Integration tests for cross-component interactions.
- E2E tests for user-facing flows (if applicable).
- Edge cases and error paths covered.
- All tests pass.

### 2c. Type safety and linting

Run the project's quality gate (from CLAUDE.md, or
`make format && make lint && make typecheck && make test`):

- Typecheck clean.
- Lint clean.
- No `any`/`unknown` types introduced without justification.

### 2d. Code consistency

- Naming, file organization, error handling, logging, and
  output formats match existing patterns in the codebase.

### 2e. Accuracy — code vs. spec

- Implementation matches the milestone spec: signatures, schemas,
  exit codes, output formats, field names, constants, thresholds.
- Where code and spec disagree, decide which is wrong. Stale spec
  → fix the spec. Code bug → fix the code. Intentional divergence
  that changes intent → surface to the operator rather than
  silently accepting.

### 2f. Integration points

- Interfaces between components are correct.
- Shared types/contracts updated consistently.
- No accidental breaking changes to existing APIs (or
  intentionally-breaking changes are documented).

## Step 3 — Documentation sync

If the project uses docs-cli, sync the docs tree:

1. Update the milestone's task plan + implementation log via body
   edits (phase section, progress-table cell, checklist tick).
2. Update `status.md`'s "Current Phase" or "Current Milestone"
   section.
3. `docs touch <slug>.md <slug>-impl.md status.md` (and any other
   docs whose body content changed this step).
4. `docs index <root>` to regenerate the marker block.
5. `docs check <root> --stale 14` — must exit 0 or 1. Exit 2 →
   stop and fix the lifecycle drift / broken refs before
   committing.

If the project does NOT use docs-cli, update whatever status /
implementation log / changelog the project's conventions require
(per CLAUDE.md).

Also update — if applicable — the README, architecture doc, API
reference, and operator runbook.

## Step 4 — Review changes

```sh
git status
git diff
```

Review every file that will be committed:

- No sensitive files staged (secrets, credentials, `.env`).
- Changes look correct and complete.
- Diff scope matches this phase/step — no unrelated changes.

## Step 5 — Commit and push

1. `cd` to the right repository (multi-repo projects commit to
   each separately).
2. `git add` the relevant files explicitly — never blanket-add
   with `git add -A` unless CLAUDE.md says otherwise.
3. Write a commit message following the project's convention
   (concise, imperative, scoped per phase when working under
   `ship-milestone`).
4. Push to remote **only** if:
   - The current branch is a feature/milestone branch (never
     `main`, never a shared branch).
   - A remote exists.
   - CLAUDE.md's branch conventions allow it.

Never use `--no-verify` or otherwise bypass git hooks unless
the operator explicitly asks.
