# Consistency / completeness / accuracy audit

The implementation agent runs this against **its own step's work**, after the
last phase of the step and before returning to the conductor. This is the
"same instance checks its own work" gate.

Use the shared risk-aware quality model at
`../../_shared/references/agentic-quality-model.md` where it exists.
Use the semantic identity, tracker, state, and relationship rules in
`../../_shared/references/milestone-tracker.md`; do not infer order or next work
from filenames, document order, or `<project-path>status.md`.
Resolve the docs root and change the working directory to it before every
docs-cli operation. `--root` does not rebase relative `FILE`, `SOURCE`, or
`TARGET` operands and is not a substitute for changing directories. From the
docs root, keep operands `<project-path>`-qualified and use `docs index .` and
`docs check .` where applicable.
Prepare the handoff under
[`review-protocol.md`](review-protocol.md); the conductor, not this audit,
selects and runs the reviewers.

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

- The owning project's canonical `<project-path>milestone-plan.md` tracker row
  is `active`; its project-unique
  semantic slug matches the milestone identity and
  `<project-branch-prefix><slug>/...` branch root. The row's linked physical
  artifact set resolves to `<project-path><artifact-stem>.md`,
  `<project-path><artifact-stem>-impl.md`, and
  `<project-path><artifact-stem>-test-matrix.md`; the stem is `<slug>` in a
  dedicated project root and a
  root-globally unique `<project>-<slug>` for new work in a shared root. Its
  Order, dependencies, and Notes have not drifted during implementation.
- `<project-path>milestone-plan.md` is the only authority for identity, order,
  state, dependencies, and derived next work. `<project-path>status.md` is a
  narrative summary that links to it; it does not duplicate tracker rows or
  independently name next.
- `<project-path><artifact-stem>.md`, its `-impl.md` and `-test-matrix.md`
  companions, `<project-path>milestone-plan.md`, and
  `<project-path>status.md` agree with reality on phase progress, test counts,
  dates, and open/resolved questions. Steps 0-2 never mark the row `complete`
  or archive the milestone artifact set; Step 3 owns that closeout.
- Materialized adjacent-order and dependency relationships match the tracker
  contract, while live blocker relationships remain distinct.
- No doc claims a behavior the code does not have, and no shipped behavior is
  undocumented.
- Cross-references and `Related:` links resolve to real files.
- Open follow-ups and feedback noted during this step live in
  `<project-path>followup-log.md` / `<project-path>feedback-log.md`, not only
  inline in milestone docs; milestone docs reference open log entries rather
  than holding them.
- Doc front-matter (`Lifecycle`, `Role`, `Updated`) is correct; any doc
  edited this step had its `Updated:` bumped per the convention (via
  `docs touch`).

## Consistency — generated artifacts

- Generated artifacts (e.g. `INDEX.md`) and any frozen test snapshots are
  regenerated **in lockstep** — byte-identical where they must match.
- `docs check .` from the resolved docs root exits 0.

## Consistency — codebase

- New code follows the existing naming, file organization, error-handling,
  logging, and output conventions.
- The milestone's taste anchors (when recorded) are honored: the code reads
  like its reference modules, in-project libraries are reused, and no new
  dependency or hand-rolled equivalent of an existing library was introduced
  without a logged decision.
- Abstraction level and comment/docstring density match the surrounding code.
- Liveness (shared quality model, Demand-driven chains): every new public
  output traces to a contract clause, a visible test, or a logged decision
  with a `<project-path>followup-log.md` ledger entry naming its intended
  consumer. Flag
  produced-but-unconsumed values — return fields no caller reads, parameters
  always passed the same value, threaded context nobody uses.
- The diff contains only this milestone's work — no unrelated changes. The
  diff footprint is in line with the plan's expected scope; note outsized
  patch bloat for the fresh-eyes reviewer rather than hiding it.
- If `explore` selected the implementation route, the implementation follows
  its recorded pattern-preservation and simplicity evidence. Any deviation is
  the smallest one supported by evidence that the existing pattern could not
  satisfy the contract; the reason is recorded without inventing a new
  approval requirement.

## Tests & quality

- The selected product test suite and configured explicit non-product checks are in the
  state the phases require: meaningful RED for new/corrected behavior or
  adequate GREEN for a pure refactor at Phase 4; fully GREEN from phase 8 onward.
- Configured lint, format check, and type check are clean for the touched
  surface, or known unrelated failures are documented.
- **Phases 1–4 only:** the product tests or explicit non-product checks
  genuinely pin the contract — they are not trivial passes, do not
  under-constrain the implementation, and do not overconstrain it by
  freezing incidental representation (byte-exact goldens or
  change-detector assertions without a contract reason). This is the
  highest-leverage check; apply the shared Check calibration rule to expected
  results and cases without adding a separate evidence record or review pass.
- New or actively modified test suites and cases have behavior-first names.
  Test-runner output states the scenario and observable behavior. Milestone,
  decision, phase, step, review, and amendment provenance stays outside display
  names; when useful, it lives in the milestone Decisions section, test matrix,
  or a nearby resolvable comment or doc. No reference syntax is required.

## Test adequacy

- Contract clauses are mapped to visible product tests or explicit
  non-product checks.
- Hidden/generalization categories are recorded without exposing private cases.
- Selected risk-level gates were run or explicitly marked not configured.
- Property/stateful, mutation, fuzz, benchmark, security, schema, migration, or
  rollback checks selected for the milestone are run, explicitly marked not
  configured, or recorded as open entries in the project's
  `<project-path>followup-log.md`.
- No code path appears keyed to visible test literals, fixture names, or narrow
  examples.
- No valid tests or selected explicit checks were weakened merely to obtain
  GREEN. Erroneous expectations were corrected under Check calibration;
  genuine contract changes and adequacy reductions followed existing policy.

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

- One commit per phase on the step branch; exploration, contract-rework, and
  WIP/blocker checkpoint commits are also allowed when their marker, evidence,
  and non-green state are explicit. Messages follow the project's convention
  (see project context and recent `git log`); no secrets staged.

## Review-ready handoff

- The return evidence names the `<project-path>milestone-plan.md` tracker path,
  owning project, active semantic
  row, claim-checkpoint commit, and semantic branch root.
- The implementation log ends this step's append-only review history with a
  uniquely headed, monotonically numbered `review-pass` entry. Its predecessor
  link is exact; after Step 1 contract rework, the immediately preceding entry
  is the newly closed `contract-rework`, while every earlier pass and frozen
  packet remains preserved. A targeted conditional re-review remains nested in
  its original pass and never substitutes for a fresh post-rework pass.
- The current `review-pass` has one Claude slot and one GPT slot, both
  initialized as `pending`. Do not infer or
  fabricate provider availability or effective model identity; the conductor
  records actual attempt outcomes later.
- The return report names the exact current pass location as the qualified
  implementation-log path plus its unique heading anchor; a general Step
  section is not a usable resume or sync location.
- The exact branch-base and review-ready `HEAD` commit SHAs, changed artifact
  paths, selected test/quality commands, and latest results needed for the
  frozen review packet are present in the return report or canonical milestone
  artifacts.
- All implementation and audit work is committed without bypassing hooks. The
  return report names the exact review-ready `HEAD`, and the working tree is
  clean so both isolated reviewers can inspect the same immutable diff.

## Return

Report: issues found and fixed (with commits), the final test + quality-gate
status, and anything surfaced for an operator decision. For each decision,
include its inspected origin/evidence, why it matters now, options and practical
effects, a grounded current-project example or clearly labeled hypothetical,
an evidence-backed recommendation or `no strong recommendation`, and the exact
operator question. This is an internal packet for the conductor, not a fixed
user-facing template.
