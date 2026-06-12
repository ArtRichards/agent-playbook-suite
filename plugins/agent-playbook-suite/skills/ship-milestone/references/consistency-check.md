# Consistency / completeness / accuracy audit

The implementation agent runs this against **its own step's work**, after the
last phase of the step and before returning to the conductor. This is the
"same instance checks its own work" gate.

Use the shared risk-aware quality model at
`../../_shared/references/agentic-quality-model.md` where it exists.

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
- Open follow-ups and feedback noted during this step live in the project's
  `followup-log.md` / `feedback-log.md` at the docs root, not only inline in
  milestone docs; milestone docs reference open log entries rather than
  holding them.
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

- The selected product test suite and configured explicit non-product checks are in the
  state the phases require: the intended RED baseline for phases 1–4; fully
  GREEN from phase 8 onward.
- Configured lint, format check, and type check are clean for the touched
  surface, or known unrelated failures are documented.
- **Phases 1–4 only:** the product tests or explicit non-product checks
  genuinely pin the contract — they are not trivial passes, do not
  under-constrain the implementation, and do not overconstrain it by
  freezing incidental representation (byte-exact goldens or
  change-detector assertions without a contract reason). This is the
  highest-leverage check; every later step trusts these tests/checks.

## Test adequacy

- Contract clauses are mapped to visible product tests or explicit
  non-product checks.
- Hidden/generalization categories are recorded without exposing private cases.
- Selected risk-level gates were run or explicitly marked not configured.
- Property/stateful, mutation, fuzz, benchmark, security, schema, migration, or
  rollback checks selected for the milestone are run, explicitly marked not
  configured, or recorded as open entries in the project's `followup-log.md`.
- No code path appears keyed to visible test literals, fixture names, or narrow
  examples.
- No tests or selected explicit checks were weakened, skipped, deleted, or
  rewritten without a logged contract change.

## Mock audit

- New or expanded mocks are justified by an external boundary, slow/paid
  service, nondeterminism, or failure-injection need.
- Each new or expanded mock records what real behavior it substitutes.
- At least one real-path test covers mocked behavior where relevant, or the
  exception is logged for operator/reviewer approval.
- Mock-heavy tests do not replace the visible contract tests for domain
  behavior.

## Hidden/generalization metrics

- `visible_pass_rate` is recorded when available.
- `hidden_pass_rate` is recorded when hidden/generalization results are
  available.
- `hidden_generalization_gap = visible_pass_rate - hidden_pass_rate` is
  recorded when both pass-rate values are available.
- Changed-lines coverage, changed-files branch coverage, mutation score on
  touched modules, property/stateful result, fuzz result, benchmark delta, and
  security/schema/migration result are recorded when available.

## Commits

- One commit per phase on the step branch; messages follow the project's
  convention (see project context and recent `git log`); no secrets staged.

## Return

Report: issues found and fixed (with commits), anything surfaced for an
operator decision, and the final test + quality-gate status.
