---
name: simplify
description: Post-implementation simplify mode for TDD Phase 10 (Quality, Docs, Refactor). Reduces code complexity while preserving behavior — replaces clever code with obvious code, removes abstraction layers that do not earn their keep, collapses needless helpers, and favors linear execution. Manually invoked (e.g. /simplify) once a milestone's implementation is complete and the product suite plus required explicit checks are green.
---

# Simplify (Post-Implementation Simplify Mode)

Use this skill during **Phase 10 (Quality, Docs, Refactor)** of the TDD process, after a
milestone's implementation is complete and the product suite plus required explicit checks
are green. It is invoked manually.

You are in post-implementation simplify mode.

Assume the current code is correct but too complex. The task is to reduce complexity while
preserving behavior.

## Establish the baseline

Before changing anything, anchor to the most recent commit:

1. Run `git log -1 --stat` and `git status` to see the last commit and what is uncommitted.
2. If a commit exists, treat its code as the **known-good output of the prior TDD phases** —
   it is correct and tested. Simplification refines that code; it must never discard,
   regress, or silently rewrite it.
3. Run `git diff HEAD` to see exactly what the current working changes are. This is the
   boundary between committed prior work and the code in flight.
4. If there is no commit yet, use the current working tree as the baseline and rely on the
   test suite alone to guard against regressions.

## Scope

Simplify the code for the current milestone/task — unless the user points to specific
files. Do not touch unrelated code, and do not undo good code from earlier TDD steps.

## Preserve quality, not only behavior

The quality baseline is part of the behavior baseline. Simplification must preserve the
contract, visible tests, hidden/generalization hooks, adequacy metrics, and realistic
coverage expected for the milestone's risk level.

Before simplifying:

- Record the visible test baseline.
- Record property/stateful, hidden smoke, mutation, fuzz, benchmark, and security
  baselines if configured for the milestone risk level.
- Record coverage and test counts if the project already reports them.
- Record new/expanded mocks and real-path coverage for mocked boundaries.
- Identify the risk level and the gate set that must still pass after simplification.

During simplification:

- Do not delete edge-case tests, property strategies, fuzz corpora, fixtures, schemas,
  or hidden-test hooks unless replacing them with clearer equivalent coverage.
- Do not collapse validation logic in a way that removes meaningful branch coverage.
- Do not replace real-path tests with mocks.
- Do not weaken contracts, reduce error handling, or remove security/schema/migration
  safeguards to make code smaller.
- Do not introduce clever abstractions merely to reduce line count.
- Do not make speculative rewrites; if the simplification cannot be explained against
  the existing contract and diff, leave the code as-is.

After simplifying:

- Run the same risk-level gate as before.
- Compare test count, coverage, mutation score, hidden/generalization result, property
  result, fuzz result, and benchmark deltas when available.
- If a metric drops, restore the stronger version or log an explicit operator-approved
  reason.
- Update the test matrix and quality log if simplification affects test or quality
  posture.

For High-risk milestones, simplification must be reviewed by a fresh-eyes reviewer or
operator before final `sync-and-commit`.

If simplification would reduce adequacy, remove realistic coverage, weaken hidden hooks,
or require speculative rewrites, return with no changes and explain why.

## Focus on

- Replacing clever code with obvious code.
- Removing abstraction layers that do not earn their keep.
- Collapsing unnecessary helpers into the caller when that makes the flow easier to follow.
- Favoring linear execution over branching where possible.
- Using the fewest concepts needed to solve the problem cleanly.

## Hard constraints

- Do not change public behavior.
- Do not add generic architecture.
- Do not create reusable abstractions unless they remove more complexity than they add.
- Do not optimize prematurely.
- Do not rewrite working code just to make it look different.
- Do not reduce visible, property, fixture, integration, hidden-hook, mutation, fuzz,
  benchmark, security, schema, or real-path coverage without explicit logged approval.
- Do not replace real-path tests with mocks or preserve a green suite by weakening tests.

## Success criteria

- A human reader should understand the code faster after the changes.
- The final version should be shorter if possible, but never at the expense of clarity.
- Every remaining line should have a clear purpose.

## Verify behavior is preserved

Behavior preservation is not optional — prove it with the relevant product tests and
required explicit checks:

1. Run the full product suite first to confirm the green baseline.
2. Run required explicit non-product checks if the milestone has them.
3. After simplifying, run the same suite and checks again. Every required gate must still pass.
4. If a test or required check fails, the change altered behavior — revert or fix it.
   Never relax a test or check to make it pass.

Also run the project quality gate, e.g. `make format && make lint && make typecheck && make test`
(or the project equivalent).

Then review the final `git diff HEAD` against the baseline: every change must be a
deliberate simplification. If the diff removes or alters code from a prior TDD step that was
not the target of this pass, restore it — that is good code being overwritten, not
simplified.

## After simplifying

- Summarize each change and why it simplifies (clever → obvious, helper collapsed, branch
  removed, abstraction dropped).
- Record the simplification in whatever implementation log the project uses. The format
  varies — a `<slug>-impl.md` milestone implementation log (the docs-cli convention used
  by `create-milestones` / `ship-milestone`), a `CHANGELOG`, a phase log, commit messages,
  or task-tracker notes. Look for the project's existing convention (project context is
  the fastest way to find out) and follow it; if none exists, skip the log rather than
  inventing one.
- Return the simplest readable version of the code.
