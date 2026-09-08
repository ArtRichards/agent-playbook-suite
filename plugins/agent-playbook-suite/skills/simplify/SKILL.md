---
name: simplify
description: Post-implementation simplify mode for TDD Phase 10 (Quality, Docs, Refactor). Reduces code complexity while preserving behavior — replaces clever code with obvious code, removes abstraction layers that do not earn their keep, collapses needless helpers, and favors linear execution. Manually invoked (e.g. /simplify) once a milestone's implementation is complete and the selected product tests plus configured explicit checks are green.
---

# Simplify (Post-Implementation Simplify Mode)

Use this skill during **Phase 10 (Quality, Docs, Refactor)** of the TDD process, after a
milestone's implementation is complete and the selected product tests plus configured
explicit checks are green. It is invoked manually.

You are in post-implementation simplify mode.

Treat the current code as the accepted baseline. The task is to reduce complexity while
preserving the agreed behavior; a commit or passing suite is evidence, not proof of correctness.

## Read first

Read [`../_shared/references/operator-interaction.md`](../_shared/references/operator-interaction.md)
before asking for a clarification, approval, or contract decision. It requires
grounded, understandable explanations without adding approval gates.

Apply the shared [Check calibration](../_shared/references/agentic-quality-model.md#check-calibration)
guidance when judging the baseline, coverage changes, or failing checks.

## Establish the baseline

Before changing anything, anchor to the most recent commit:

1. Run `git log -1 --stat` and `git status` to see the last commit and what is uncommitted.
2. If a commit exists, treat its code as **accepted prior work**. Check the available
   regression evidence; simplification must not discard that work or silently change
   its agreed behavior.
3. Run `git diff HEAD` to see exactly what the current working changes are. This is the
   boundary between committed prior work and the code in flight.
4. If there is no commit yet, use the current working tree as the baseline and judge
   regression evidence against the contract.

## Scope

Simplify the code for the current milestone/task — unless the user points to specific
files. Do not touch unrelated code, and do not undo good code from earlier TDD steps.

## Preserve quality, not only behavior

The quality baseline is part of the behavior baseline. Simplification must preserve the
contract, meaningful protection from selected tests and hidden/generalization hooks,
and realistic coverage expected for the milestone's risk level. Metrics already being
tracked help assess that protection.

Before simplifying:

- Record the visible test baseline.
- Record property/stateful, hidden smoke, mutation, fuzz, benchmark, and security
  baselines if configured for the milestone risk level.
- Record coverage and test counts if the project already reports them.
- Record new/expanded mocks and real-path coverage for mocked boundaries.
- Identify the risk level and the selected gate set that must still pass after
  simplification.

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

- Verify the same selected risk-level gates as described below.
- Compare test count, coverage, mutation score, hidden/generalization result, property
  result, fuzz result, and benchmark deltas when available.
- If a metric drops, assess whether meaningful protection was lost. A denominator
  change or removal of duplicate tests with equivalent coverage is not itself a
  regression. Restore lost protection or request the existing explicit
  operator-approved exception, explaining the before/after protection and why
  accepting that loss is needed now.
- Update the test matrix and quality log if simplification affects test or quality
  posture.

For High-risk milestones, when simplification changes code, freeze the exact candidate
and obtain approval before committing it or final sync: use one fresh-eyes reviewer
when available, otherwise ask the operator under the shared interaction policy. In
`ship-milestone`, this happens before the clean pre-archive checkpoint. This is a
single conditional gate, not a two-provider review. Record the reviewer/operator,
frozen tree id, findings and dispositions; with no code change, record `not required`.

If simplification would require speculative rewrites or leave an unapproved loss of
meaningful protection, return with no changes and explain why.

## Focus on

- Replacing clever code with obvious code.
- Removing abstraction layers that do not earn their keep.
- Collapsing unnecessary helpers into the caller when that makes the flow easier to follow.
- Favoring linear execution over branching where possible.
- Using the fewest concepts needed to solve the problem cleanly.
- Aligning with the project's observed practices: prefer libraries already in
  the project over new dependencies or hand-rolled equivalents; converge on
  the codebase's established idioms; match the surrounding naming, structure,
  and comment/docstring density.
- Removing dead information flow: return fields no caller reads, parameters
  always passed the same value, threaded context nobody consumes, results
  computed and then dropped. If a test asserts an output, resolve its contract
  basis under Check calibration before removing it. When the dead output is part of the contract,
  do NOT remove it unilaterally — surface a
  contract-change proposal instead; removal goes through the "contract
  changed and the decision is logged" path. Dead outputs covered by a
  speculative-ledger entry in `<project-path>followup-log.md` stay: their
  consumer is a named future milestone.
- Addressing waived or deferred taste findings from review when a
  behavior-preserving change fixes them — then update their taste-triage
  entries (in the impl log) to fixed. Do not expand scope to chase findings
  that need behavior changes; those stay waived.

## Hard constraints

- Do not change public behavior.
- Do not add generic architecture.
- Do not create reusable abstractions unless they remove more complexity than they add.
- Do not optimize prematurely.
- Do not rewrite working code just to make it look different.
- Do not reduce meaningful protection from selected visible, property, fixture,
  integration, hidden-hook, mutation, fuzz, benchmark, security, schema, or
  real-path checks without explicit logged approval.
- Do not replace real-path tests with mocks or preserve a green suite by weakening tests.

## Success criteria

- A human reader should understand the code faster after the changes.
- The final version should be shorter if possible, but never at the expense of clarity.
- Every remaining line should have a clear purpose.

## Verify behavior is preserved

Use the same selected product tests and configured explicit checks to establish
regression evidence. Reuse recorded results when their relevant code, tests,
configuration, and environment are unchanged, unless project policy explicitly
requires a fresh run:

1. Establish a GREEN baseline for the selected product suite or focused gate.
2. Include configured explicit non-product checks if the milestone has them.
3. After simplifying, rerun checks affected by the change. Every selected gate must
   have a still-applicable passing result.
4. If a check fails, diagnose the behavior, expectation, or setup against the
   contract before changing it. Revert or fix a regression; correct a faulty
   expectation or setup without weakening the agreed contract to obtain GREEN.

Include the configured project quality gate, or the relevant subset for the touched
files, e.g. `make format && make lint && make typecheck && make test` (or the
project equivalent), reusing applicable results under the same rule.

Then review the final `git diff HEAD` against the baseline: every change must be a
deliberate simplification. If the diff removes or alters code from a prior TDD step that was
not the target of this pass, restore it as outside this simplification's scope.

## After simplifying

- Summarize each change and why it simplifies (clever → obvious, helper collapsed, branch
  removed, abstraction dropped).
- Record the simplification in whatever implementation log the project uses. The format
  varies — a `<project-path><artifact-stem>-impl.md` milestone implementation log
  (the docs-cli convention used by `create-milestones` / `ship-milestone`), a
  `CHANGELOG`, a phase log, commit messages, or task-tracker notes. For suite-managed
  work, resolve both values from `<project-path>milestone-plan.md`; semantic `<slug>` is
  tracker and branch identity, not a physical filename operand. Look for the project's
  existing convention (project context is the fastest way to find out) and follow it; if
  none exists, skip the log rather than inventing one.
- When updating a docs-cli-managed log, resolve the docs root and change the working
  directory to it before any `docs touch` or other docs-cli operation. `--root` does not
  rebase relative `FILE`, `SOURCE`, or `TARGET` operands and is not a substitute for
  changing directories. Use the root-relative `<project-path>`-qualified operand, then
  run `docs index .` and `docs check .` from that docs root.
- Return the simplest readable version of the code.
