# TDD Phases — 10-Phase Reference

The canonical phase definitions for milestone-driven TDD work.
The orchestration that drives them lives in
[`milestone-playbook.md`](milestone-playbook.md).

Each phase below lists: objective, activities, deliverables, exit
criteria, and the docs-CLI touchpoints (every phase ends with at
least a `docs touch` on the impl log).

Use the shared quality model at
[`../../_shared/references/agentic-quality-model.md`](../../_shared/references/agentic-quality-model.md)
for risk levels, hidden/generalization policy, adequacy checks,
and mock policy.

## Cross-phase quality policies

- Visible tests drive implementation, but green visible tests are
  not sufficient evidence of intent for Standard or High-risk work.
- Classify validation as product tests or explicit non-product
  checks. Planning, documentation, and handoff checks must not enter
  default product-test discovery unless they define shipped behavior.
- Actual hidden/private cases must not be pasted into milestone
  docs, visible tests, implementation prompts, or implementation
  context. Record categories, owners, command names, and summaries
  instead.
- If the repository is fully visible to the implementation agent,
  assume hidden cases are not truly hidden and choose additional
  property/stateful, mutation, fuzz, metamorphic, benchmark, or
  review-agent checks only when they fit the risk.
- Do not add or expand mocks without justification. Prefer
  real-path tests for domain behavior, and record whether a
  real-path test covers each mocked boundary.
- Do not weaken, skip, delete, or narrow tests or configured explicit
  checks unless the contract changes and the decision is logged.
- Do not special-case visible examples, fixture names, literals, or
  test-only branches.
- Deferred work and feedback have a single home in the project logs
  at the docs root: engineering deferrals go to `followup-log.md`,
  operator feedback and ideas to `feedback-log.md` (both created by
  `project-foundation`, with entry templates embedded in each).
  Milestone docs reference open entries while they remain open and
  drop the reference once incorporated. Do not leave open items only
  in milestone docs — those archive at completion and the item is
  lost.

## Phase 1 — Define Contract

- **Objective:** Specify the behavior contract before tests or
  implementation: intended behavior, scope, facts, assumptions,
  ambiguities, inputs, outputs, invariants, error cases,
  non-functional constraints, acceptance examples, forbidden
  shortcuts, test hooks, and risk level.
- **Activities:** Fill the milestone doc's `Contract`, `Risk
  Level`, `Test Strategy For This Milestone`, and initial `Test
  Matrix` sections; separate facts from assumptions; resolve or
  mark ambiguities for operator decision; identify visible test
  hooks, hidden/generalization categories, adequacy checks, and
  mock policy.
- **Deliverables:** Behavior contract recorded; risk level assigned;
  test matrix created or linked; public inputs, outputs,
  invariants, error cases, and non-functional constraints made
  testable; no business logic yet.
- **Exit:**
  - [ ] Behavior contract exists.
  - [ ] Facts and assumptions are separated.
  - [ ] Ambiguities are resolved or marked for operator decision.
  - [ ] Public inputs, outputs, invariants, and error cases are testable.
  - [ ] Non-functional constraints are recorded or explicitly marked not applicable.
  - [ ] Risk level is assigned.
  - [ ] Test matrix has been created or linked.
- **Docs touchpoints:**
  - Append a Phase 1 section to `<slug>-impl.md` body.
  - Update `<slug>-test-matrix.md` with initial contract clauses
    and planned validation layers.
  - Tick `[x] Phase 1` in `<slug>.md`'s checklist; flip the
    progress-table cell from `Pending` to `Complete`.
  - `docs touch <slug>.md <slug>-impl.md <slug>-test-matrix.md
    status.md`.
  - `docs check <root> --stale 14` exit 0 or 1.

## Phase 2 — Write Tests (RED)

- **Objective:** Express desired behaviour as failing product tests
  or explicit non-product checks before implementation, using enough
  evidence to pin the contract without overbuilding the test surface.
- **Activities:** Write visible product tests that trace to contract
  clauses where practical and, when the project has a
  `use-cases.md`, to the primary use cases they demonstrate —
  tests focus on use cases first; for planning, documentation, or handoff
  artifacts, write explicit checks outside default product-test
  discovery. Prefer behavior tests over implementation-detail tests;
  prefer the least constraining check that gives real confidence in
  the intended behavior — check semantic behavior first and do not
  freeze incidental representation (byte-exact goldens, exhaustive
  snapshots, change-detector assertions) unless the exact
  representation is itself the contract;
  add negative and boundary cases where they clarify the contract or
  cover meaningful risk; propose hidden/generalization categories
  separately; justify any new mocks.
- **Deliverables:** Test or check module(s) with clear names and
  docstrings; minimum coverage targets noted where applicable; test
  matrix updated with visible tests/checks and hidden/generalization
  categories.
- **Exit:**
  - [ ] Each changed public behavior is pinned by a visible product
        test, explicit check, or logged acceptance rationale.
  - [ ] Each selected non-product gate has an explicit check.
  - [ ] Negative and boundary cases exist where applicable.
  - [ ] Checks constrain intended behavior, not incidental
        representation (no byte-exact goldens or change-detector
        assertions without a contract reason).
  - [ ] Tests are expected to fail for missing behavior, not import/setup mistakes.
  - [ ] Hidden/generalization categories are recorded without leaking private cases.
  - [ ] New mocks are listed and justified.
- **Docs touchpoints:** standard per-phase set above.

## Phase 3 — Create Data/Fixtures

- **Objective:** Provide synthetic data/builders to exercise the
  tests.
- **Activities:** Add representative, boundary, malformed,
  adversarial, and realistic fixtures where applicable; cover all
  enums / statuses / variants; ensure correctness for money,
  date-handling, edge sizes; add at least one real-path fixture
  where mocks are used.
- **Deliverables:** Fixture files or factory helpers; integrity
  checked.
- **Exit:** Data loads cleanly; represents every required variant;
  hidden cases are not encoded into visible fixtures; no fixture
  exists only to satisfy a narrow visible assertion.
- **Docs touchpoints:** standard per-phase set above.

## Phase 4 — Run Tests (RED Baseline)

- **Objective:** Confirm tests fail for the right reasons.
- **Activities:** Run the focused product tests and any explicit
  non-product checks selected for this phase; capture failure summary
  verbatim; inspect already-green visible tests/checks; check that
  the test matrix is complete enough to proceed.
- **Deliverables:** Test/check output with failure reasons noted
  in the impl log; RED status summarized in the test matrix when
  useful.
- **Exit:** Failing tests/checks trace to missing implementation, not
  misconfiguration; the test matrix is complete enough to proceed.
  Trivial, under-constrained, already-green, or setup-only tests/checks
  block progression only when they are the main evidence for a
  contract clause.
- **Docs touchpoints:** standard per-phase set above. Paste the
  RED baseline output into the Phase 4 log section verbatim.
- **High-risk checkpoint:** For High-risk milestones, stop after
  the RED baseline and ask the operator to approve the contract,
  visible tests, hidden/generalization plan, mock policy, and risk
  selected gates before Phase 5 starts, unless project policy explicitly
  allows automatic continuation.

## Phase 5 — Update Base Interfaces

- **Objective:** Adjust shared interfaces / base classes for new
  parameters or behaviours.
- **Activities:** Add method signatures, shared utilities,
  validation hooks.
- **Deliverables:** Updated base modules; minimal logic, just
  the scaffolding to support later phases.
- **Exit:** Type checks pass; downstream components can import
  the new interfaces.
- **Docs touchpoints:** standard per-phase set above. If a
  decision was made here (e.g., a base-class API choice), append
  an entry to `decision-log.md` and `docs touch` it.

## Phase 6 — Implement Offline/Core Path

- **Objective:** Make the offline / local / core implementation
  pass tests.
- **Activities:** Implement filtering, pagination, business
  rules; keep changes scoped.
- **Deliverables:** Offline/core provider or module implemented;
  helpers factored.
- **Exit:** Target tests passing in offline/core mode; no
  regressions in existing suites.
- **Docs touchpoints:** standard per-phase set above.

## Phase 7 — Update Tool/Wrapper Layer

- **Objective:** Validate inputs/outputs at the boundary (tool
  wrappers, controllers, RPC handlers).
- **Activities:** Add request/response validation; wire
  providers; enforce enums/ranges.
- **Deliverables:** Wrapper updated; errors/user-facing
  messages aligned with the contract.
- **Exit:** Lint and type checks pass; wrappers delegate to the
  correct provider paths.
- **Docs touchpoints:** standard per-phase set above. If this
  phase changes user-visible CLI or API surface, append a note
  to the relevant spec doc (`scope-and-constraints.md` or a
  dedicated API reference) and `docs touch` it.

## Phase 8 — Run Tests (GREEN)

- **Objective:** Achieve a passing state for the implemented
  path(s) and satisfy the risk-appropriate gate.
- **Activities:** Run focused suites; iterate fixes; keep a
  changelog in the impl log; run the selected product gates and
  explicit non-product checks for the milestone's Risk Level.
- **Deliverables:** Passing test/check output; notes on any flaky cases;
  coverage / property / hidden smoke / mock audit / security /
  schema / benchmark / mutation summaries where required.
- **Exit:** Selected gates for the Risk Level are green or
  explicitly approved as skipped:
  - **Lite:** visible product tests green; explicit non-product
    checks green when selected; lint/type/build green where
    configured; docs check green.
  - **Standard:** Lite plus coverage report if configured;
    property/stateful smoke where useful; hidden/generalization
    smoke where configured; mock audit complete.
  - **High:** Standard plus security/schema/benchmark/migration
    checks where they match the risk; mutation smoke or baseline where
    configured; reviewer/operator signoff where required.
- **Docs touchpoints:** standard per-phase set above. Paste
  GREEN test/check output verbatim into the Phase 8 log section. If
  anything is RED, STOP — fix the root cause; do not relax a
  test.

## Phase 9 — Integrate / Accept / Dogfood

- **Objective:** Validate the change in realistic product,
  integration, or operator context.
- **Activities:** Depending on project type, run API integration,
  UI browser checks, CLI dogfooding, benchmark validation,
  migration rehearsal, realistic fixture runs, or manual
  exploratory acceptance. Add online / remote paths only when the
  milestone actually owns them.
- **Deliverables:** Integration or acceptance results; realistic
  fixture metrics; benchmark or migration evidence where relevant;
  clear TODOs for blocked external dependencies.
- **Exit:** The milestone's user-visible or integration behavior
  is accepted for its Risk Level, or blocked items are documented
  with owner and risk and recorded as open `followup-log.md`
  entries.
- **Docs touchpoints:** standard per-phase set above. Many
  milestones have no online surface; in that case repurpose
  Phase 9 for dogfooding against realistic fixtures and
  document the pass criteria + measured metrics in the Phase 9
  log section.

## Phase 10 — Quality, Docs, Refactor

- **Objective:** Final polish and handoff readiness.
- **Activities:**
  - Run the project quality gate or the relevant configured subset
    (`make format && make lint && make typecheck && make test` or
    project equivalent).
  - Update the test matrix.
  - Summarize adequacy results.
  - Record hidden-generalization gap if visible and hidden pass
    rates are available.
  - Log mutation/property/fuzz/benchmark gaps as open
    `followup-log.md` entries if not run.
  - Complete the mock audit.
  - Refactor for clarity (the `simplify` skill may help here).
  - Confirm simplification did not reduce selected test adequacy
    without explicit approval.
  - Update user-facing docs.
  - Append a milestone-completion summary to both the milestone
    doc and the impl log.
- **Deliverables:** Updated documentation set; implementation
  checklist completed; test matrix current; adequacy results and
  follow-up gaps recorded; open questions noted; release notes if
  applicable.
- **Exit:** Selected quality gate green; documentation current;
  adequacy results summarized; mock audit complete when relevant;
  skipped selected deep gates have explicit approval or an open
  `followup-log.md` entry; handoff notes written. Ready for
  `docs archive <slug>.md --cascade --reason "<reason>"`.
- **Docs touchpoints:**
  - Append "Milestone-completion summary" sections to both
    `<slug>.md` and `<slug>-impl.md`.
  - Update `<slug>-test-matrix.md` with final matrix and adequacy
    results.
  - `docs touch` all milestone docs.
  - `docs check <root> --stale 14` must exit 0 or 1.
  - **Do not archive yet** — the playbook's Step 4 owns the
    archive call so the project-level status update happens in
    one place.

## Per-phase docs hygiene checklist

After every phase, regardless of which one:

- [ ] Phase section appended to `<slug>-impl.md` body (objective,
      files, actions, results, decisions).
- [ ] `<slug>-test-matrix.md` updated when contract clauses,
      visible tests, hidden/generalization categories, adequacy
      checks, or mock policy changed.
- [ ] Progress-table cell in `<slug>-impl.md` flipped to
      `Complete`.
- [ ] `[x]` ticked in `<slug>.md`'s Phase Checklist.
- [ ] `status.md`'s "Current Phase" line updated.
- [ ] `docs touch <slug>.md <slug>-impl.md
      <slug>-test-matrix.md status.md`.
- [ ] `docs index <root>` regenerated.
- [ ] `docs check <root> --stale 14` exit 0 or 1.
- [ ] User confirmation before starting the next phase.

## Phase ordering invariants

- **Phases run sequentially.** Never skip ahead.
- **Phase 4 must reveal RED for the right reasons** — a Phase 4
  that already shows GREEN means Phase 2 didn't test what was
  intended; go back.
- **High-risk milestones pause after Phase 4** for operator
  approval of the contract, visible tests, hidden/generalization
  plan, mock policy, and selected gates unless project policy
  explicitly allows automatic continuation.
- **Phase 8 selected gates must be GREEN before Phase 9.** A flaky
  selected test is not GREEN; either fix the flake or scope it out
  with a documented TODO before proceeding.
- **Risk gates are cumulative by default.** Lite gates are the base
  for Standard; Standard gates are the base for High unless project
  policy selects a lighter equivalent or marks a gate not applicable.
- **Phase 10 always runs**, even for milestones that feel small.
  The completion summary, test matrix, adequacy results, and docs
  touch are what make archive meaningful.

## When a phase reveals the plan was wrong

If during a phase you discover the milestone plan needs
substantial revision:

1. Stop the phase. Do not press on with the wrong plan.
2. Append a decision-log entry capturing what was discovered and
   what needs to change.
3. Edit the milestone doc's relevant Phase section and Phase
   Checklist to reflect the new shape.
4. Update `<slug>-test-matrix.md` if contract clauses, visible
   tests, hidden/generalization categories, adequacy checks, or
   mock policy changed.
5. `docs touch <slug>.md <slug>-test-matrix.md` and the decision
   log.
6. Resume the (now corrected) phase.

Plan revision is normal — every long-running milestone has at
least one. The audit trail is what makes it safe.
