# Consistency / completeness / accuracy audit

The implementation agent runs this against **its own step's work**, after the
last phase of the step and before returning to the conductor. This is the
"same instance checks its own work" gate.

Work through every item. **Fix** what you find and commit the fixes. **Surface**
(do not auto-decide) anything that would change milestone scope or behavior
intent — list those in your return message for the operator.

## Completeness

- Every Deliverable and Success Criterion that falls in this step's phase range
  is genuinely met — verified against the milestone doc, not from memory.
- No placeholder code, no `NotImplementedError` stubs for phases that are meant
  to be shipped, no leftover TODOs, no commented-out code.
- Every phase in range has an accurate entry in the milestone log; the phase
  table is filled with status and date.

## Accuracy — code vs. spec

- The implementation matches every pinned spec the milestone references
  (e.g. `cli.md`, `architecture.md`, `convention.md`): signatures, JSON/data
  schemas, exit codes, output formats, field names, rule/enum ids, constants,
  thresholds, limits.
- Where code and spec disagree, decide which is wrong. A stale/incorrect spec
  is corrected to match the shipped behavior; a code bug is fixed. If the
  divergence is intentional and changes intent, surface it instead.

## Consistency — documentation

- `status.md`, the milestone doc, the milestone log, and `milestone-plan.md`
  agree with each other and with reality: phase progress, test counts, dates,
  "next" pointers, open/resolved questions.
- No doc claims a behavior the code does not have, and no shipped behavior is
  undocumented.
- Cross-references and `Related:` links resolve to real files.
- Doc front-matter (`Lifecycle`, `Role`, `Updated`) is correct; any doc
  edited this step had its `Updated:` bumped per the convention (via
  `docs touch`).

## Consistency — generated artifacts

- Generated artifacts (e.g. `INDEX.md`) and any frozen test snapshots are
  regenerated **in lockstep** — byte-identical where they must match.
- `docs check` on the project's docs tree exits 0.

## Consistency — codebase

- New code follows the existing naming, file organization, error-handling,
  logging, and output conventions.
- The diff contains only this milestone's work — no unrelated changes.

## Tests & quality

- The suite is in the state the phases require: the intended RED baseline for
  phases 1–4; fully GREEN from phase 8 onward.
- Lint, format check, and type check are clean tree-wide.
- **Phases 1–4 only:** the tests genuinely pin the contract — they are not
  trivial passes and do not under-constrain the implementation. This is the
  highest-leverage check; every later step trusts these tests.

## Commits

- One commit per phase on the step branch; messages follow the project's
  convention (see `CLAUDE.md` and recent `git log`); no secrets staged.

## Return

Report: issues found and fixed (with commits), anything surfaced for an
operator decision, and the final test + quality-gate status.
