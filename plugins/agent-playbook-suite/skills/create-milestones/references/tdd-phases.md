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

Use the shared milestone tracker contract at
[`../../_shared/references/milestone-tracker.md`](../../_shared/references/milestone-tracker.md)
for semantic identity, tracker state, dependencies, recognized
relationships, and derived next work. Phase execution starts only after
the selected row is `active`; `<status-path>` never substitutes for the
tracker.

The caller resolves the path vocabulary in `milestone-playbook.md` before any
phase: `<project-path>`, `<milestone-path>`, `<impl-path>`, `<matrix-path>`,
`<tracker-path>`, `<status-path>`, `<archive-scope>`, and `<docs-root>`. Run
every displayed docs-cli command from `<docs-root>`. `<slug>` remains semantic
tracker identity only; never derive a physical filename from it inside a
phase.

## Cross-phase quality policies

- Visible tests drive implementation; complete the selected risk-level gates
  before acceptance. Use the shared model's Check calibration guidance for
  meaningful cases, failure diagnosis, and when to stop expanding testing.
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
- Do not weaken, skip, delete, or narrow valid tests or configured explicit
  checks merely to obtain GREEN. Correct erroneous checks under the shared
  model's Check calibration rule; preserve the agreed behavior and coverage.
- Do not special-case visible examples, fixture names, literals, or
  test-only branches.
- Taste is part of the quality model (see the shared quality model's
  Taste model): taste anchors are recorded at Phase 1, honored during
  implementation, and every taste finding raised in review is triaged —
  fixed, or waived with a logged reason — before the work is finalized.
  Never dropped silently.
- Record the demand, not the implementation (see the shared quality
  model's Demand-driven chains): when work reveals a future need outside
  the milestone's Deliverables, record it in the milestone plan or
  `<project-path>followup-log.md` — do not build surface for it. A public output
  neither named in the contract nor asserted by a visible test is
  speculative and needs a logged decision plus a ledger entry.
- Deferred work and feedback have a single home in the project logs
  in the owning project: engineering deferrals go to
  `<project-path>followup-log.md`, operator feedback and ideas to
  `<project-path>feedback-log.md` (both created by
  `project-foundation`, with entry templates embedded in each).
  Milestone docs reference open entries while they remain open and
  drop the reference once incorporated. Do not leave open items only
  in milestone docs — those archive at completion and the item is
  lost.
- Preserve the active semantic slug throughout all phases. A phase may
  refine the milestone title or scope, but never rename its activated
  identity or infer execution order from filenames. If evidence changes
  durable prerequisites, update `Depends on` and synchronize
  `depends-on` / `required-by` through the tracker workflow before the
  phase proceeds; transient blockers use `blocks` / `blocked-by` and do
  not create a stored `blocked` state.

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
  mock policy; record taste anchors per the shared quality model
  (reference modules the change should read like, in-project
  libraries to reuse, established patterns, expected diff
  footprint, dependency policy).
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
  - [ ] Taste anchors are recorded or explicitly marked not applicable.
  - [ ] Risk level is assigned.
  - [ ] Test matrix has been created or linked.
- **Docs touchpoints:**
  - Append a Phase 1 section to `<impl-path>` body.
  - Update `<matrix-path>` with initial contract clauses
    and planned validation layers.
  - Tick `[x] Phase 1` in `<milestone-path>`'s checklist; flip the
    progress-table cell from `Pending` to `Complete`.
  - `docs touch <milestone-path> <impl-path> <matrix-path>
    <status-path>`.
  - `docs check . --stale 14` exit 0 or 1.

## Phase 2 — Write Tests (RED)

- **Objective:** Express desired behaviour as failing product tests
  or explicit non-product checks before implementation, using enough
  evidence to pin the contract without overbuilding the test surface.
- **Activities:** Reuse or write visible product tests that trace to contract
  clauses where practical and, when the project has a
  `<project-path>use-cases.md`, to the primary use cases they demonstrate —
  tests focus on use cases first; for planning, documentation, or handoff
  artifacts, write explicit checks outside default product-test
  discovery. Prefer behavior tests over implementation-detail tests;
  prefer the least constraining check that gives real confidence in
  the intended behavior — check semantic behavior first and do not
  freeze incidental representation (byte-exact goldens, exhaustive
  snapshots, change-detector assertions) unless the exact
  representation is itself the contract; name new and actively
  modified test suites and cases for the scenario and observable
  behavior in plain domain language so test-runner output stands
  alone; keep milestone, decision, phase, step, review, and amendment
  provenance outside display names and, when useful, place it in the
  milestone Decisions section, test matrix, or a nearby resolvable
  comment or doc; do not require repository-wide renaming;
  choose cases that reject a plausible wrong result under the shared model's
  Check calibration rule; add negative and boundary cases where they clarify
  the contract or cover meaningful risk; propose hidden/generalization categories
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
  - [ ] New and actively modified test names make the scenario and
        observable behavior clear from test-runner output alone; any
        useful provenance lives outside the display name and resolves
        without guessing.
  - [ ] New or corrected behavior has tests expected to fail for that reason,
        not import/setup mistakes. For a behavior-preserving refactor, an
        adequate GREEN baseline suffices; add tests only for an actual gap.
  - [ ] Hidden/generalization categories are recorded without leaking private cases.
  - [ ] New mocks are listed and justified.
- **Docs touchpoints:** standard per-phase set above.

## Phase 3 — Create Data/Fixtures

- **Objective:** Provide synthetic data/builders to exercise the
  tests.
- **Activities:** Add representative, boundary, malformed,
  adversarial, and realistic fixtures where applicable; cover the
  enums / statuses / variants required by the changed behavior; ensure correctness for money,
  date-handling, edge sizes; add at least one real-path fixture
  where mocks are used.
- **Deliverables:** Fixture files or factory helpers; integrity
  checked.
- **Exit:** Data loads cleanly; represents every required variant;
  hidden cases are not encoded into visible fixtures; no fixture
  exists only to satisfy a narrow visible assertion.
- **Docs touchpoints:** standard per-phase set above.

## Phase 4 — Run Tests (RED Baseline)

- **Objective:** Confirm the intended baseline: RED for new or corrected
  behavior; adequate GREEN coverage for a behavior-preserving refactor.
- **Activities:** Run the focused product tests and any explicit
  non-product checks selected for this phase; capture results
  verbatim; inspect already-green visible tests/checks; check that
  the test matrix is complete enough to proceed.
- **Deliverables:** Test/check output with the expected state and its reason
  in the existing impl log; baseline status summarized in the test matrix when
  useful.
- **Exit:** Failing tests/checks trace to missing implementation, not
  misconfiguration; the test matrix is complete enough to proceed.
  Already-green checks are valid for supported behavior and pure refactors.
  An unexpected pass for missing behavior, or trivial, under-constrained, or
  setup-only checks used as the main clause evidence, blocks progression.
- **Docs touchpoints:** standard per-phase set above. Paste the
  baseline output into the Phase 4 log section verbatim.
- **High-risk checkpoint:** For High-risk milestones, stop after
  the RED baseline and ask the operator to approve the contract,
  visible tests, hidden/generalization plan, mock policy, and risk
  selected gates before Phase 5 starts, unless project policy explicitly
  allows automatic continuation. This checkpoint also applies to a High-risk
  refactor with a GREEN baseline.
- **Solution-uncertainty checkpoint:** Risk and solution uncertainty are
  separate. Before Phase 5, invoke `explore` only when the shared quality
  model's automatic-trigger gates are met after direct inspection or a cheap
  probe. Keep the route registry and evidence in the implementation log; record
  only the disposition and link in the milestone's Decisions section. A
  selected route informs Phases 5-7; an
  operator-owned product decision is surfaced with evidence and tradeoffs; no
  viable route or an exact unresolved gap stops progression. Exploration never
  changes intended behavior or fixed constraints. If it identifies a contract
  or constraint change as the only unlock, return that decision to the contract
  owner. Only after the owner explicitly approves the change, return to Phase 1
  and update the contract and RED tests rather than treating it as an
  implementation choice.

## Phase 5 — Update Base Interfaces

- **Objective:** Adjust shared interfaces / base classes for new
  parameters or behaviours.
- **Activities:** Add method signatures, shared utilities,
  validation hooks. Design pull-style: derive interfaces backward
  from the milestone's Deliverables and the Phase 2 tests, not
  forward from guesses about what later phases might want. Each
  intermediate output (return field, parameter, threaded context)
  names its downstream consumer; anything speculative is a logged
  decision with a `<project-path>followup-log.md` ledger entry naming the
  intended future consumer.
- **Deliverables:** Updated base modules; minimal logic, just
  the scaffolding to support later phases.
- **Exit:** Type checks pass; downstream components can import
  the new interfaces; no unlogged speculative surface.
- **Docs touchpoints:** standard per-phase set above. If a
  decision was made here (e.g., a base-class API choice), append
  an entry to `<project-path>decision-log.md` and `docs touch` it.

## Phase 6 — Implement Offline/Core Path

- **Objective:** Make the offline / local / core implementation
  pass tests.
- **Activities:** Implement filtering, pagination, business
  rules; keep changes scoped. Implement to the Phase 1 taste
  anchors: a dependency, hand-rolled equivalent of an in-project
  library, or pattern the anchors do not cover is a logged
  decision, not a silent choice.
- **Deliverables:** Offline/core provider or module implemented;
  helpers factored.
- **Exit:** Target tests passing in offline/core mode; no
  regressions in existing suites.
- **Docs touchpoints:** standard per-phase set above.

## Phase 7 — Update Tool/Wrapper Layer

- **Objective:** Validate inputs/outputs at the boundary (tool
  wrappers, controllers, RPC handlers).
- **Activities:** Add request/response validation; wire
  providers; enforce enums/ranges. Follow the taste anchors at
  the boundary layer (error shapes, message style, validation
  patterns already in use).
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
    property/stateful smoke selected for the contract; hidden/generalization
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
  milestone actually owns them. Integration checks may run earlier when useful;
  this phase confirms acceptance and also runs the liveness walk: trace
  backward from the observable outputs and flag anything the
  chain produces but never consumes — return fields no caller
  reads, parameters always passed the same value, threaded
  context nobody uses. Each flag is fixed, or recorded as a
  minimality finding for taste triage / the simplify pass.
- **Deliverables:** Integration or acceptance results; realistic
  fixture metrics; benchmark or migration evidence where relevant;
  clear TODOs for blocked external dependencies.
- **Exit:** The milestone's user-visible or integration behavior
  is accepted for its Risk Level, or blocked items are documented
  with owner and risk and recorded as open `<project-path>followup-log.md`
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
  - Log gaps in selected mutation/property/fuzz/benchmark checks as open
    `<project-path>followup-log.md` entries if not run.
  - Complete the mock audit.
  - Run the taste checklist from the shared quality model's Taste
    model against the milestone diff; record patch bloat (diff
    size vs. the plan's expected footprint).
  - Triage every open taste finding: fixed, or waived with a
    logged reason. High-risk waivers need operator approval.
    Note recurring findings/waivers for promotion into project
    conventions (CLAUDE.md / AGENTS.md) or future taste anchors.
  - Run the boundary sweep from the shared quality model's
    Demand-driven chains: every speculative output this milestone
    leaves behind gets a `<project-path>followup-log.md` ledger entry naming its
    intended consumer, or is removed now; ledger entries naming
    THIS milestone as consumer are closed (consumed) or challenged
    (the reserved surface went unused — remove it or re-justify).
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
  taste findings triaged (each fixed or waived with a logged
  reason — none untriaged); boundary sweep complete (speculative
  outputs ledgered or removed; inbound ledger entries closed or
  challenged); skipped selected deep gates follow the shared risk policy,
  with deferred work in `<project-path>followup-log.md`; handoff notes written; the
  milestone tracker row remains `active`, links the materialized task
  plan, and matches its durable dependencies and recognized reciprocal
  relationships. Ready for the playbook's same-scope preview and
  `docs archive <milestone-path> --cascade-only '<archive-scope>' --reason
  "<reason>"`, where the frozen scope is
  `<project-path><artifact-stem>-*`.
- **Docs touchpoints:**
  - Append "Milestone-completion summary" sections to both
    `<milestone-path>` and `<impl-path>`.
  - Update `<matrix-path>` with final matrix and adequacy
    results.
  - `docs touch` all milestone docs.
  - `docs check . --stale 14` must exit 0 or 1.
  - **Do not archive yet** — the playbook's Step 4 owns the
    archive call so the project-level status update happens in
    one place.

## Per-phase docs hygiene checklist

After every phase, regardless of which one:

- [ ] Phase section appended to `<impl-path>` body (objective,
      files, actions, results, decisions).
- [ ] `<matrix-path>` updated when contract clauses,
      visible tests, hidden/generalization categories, adequacy
      checks, or mock policy changed.
- [ ] Progress-table cell in `<impl-path>` flipped to
      `Complete`.
- [ ] `[x]` ticked in `<milestone-path>`'s Phase Checklist.
- [ ] `<status-path>`'s current milestone/phase narrative updated without adding a
      second tracker or stored next-work choice.
- [ ] `docs touch <milestone-path> <impl-path> <matrix-path>
      <status-path>`.
- [ ] `docs index .` regenerated.
- [ ] `docs check . --stale 14` exit 0 or 1.
- [ ] User confirmation before starting the next phase.

## Phase ordering invariants

- **Phases run sequentially.** Never skip ahead.
- **Phase 4 must establish the intended baseline** — new or corrected behavior
  requires meaningful RED; a pure refactor may use adequate GREEN coverage.
  Do not manufacture failures or add mutation checks to satisfy this phase.
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
2. If concrete evidence blocked or invalidated the selected technical route,
   invoke `explore` in recovery mode before revising the plan. Do not repeat the
   failed mechanism; consume the existing route record and reopen it only under
   the shared quality model's reopen rules.
3. Append a decision-log entry capturing what was discovered and
   what needs to change.
4. If the revision changes tracker-owned order, state, dependencies,
   or blockers, apply the shared tracker workflow and synchronize the
   recognized relationship pairs before continuing. The activated
   semantic slug is frozen: change the title for wording refinements,
   or cancel/supersede and create a new row for a genuine identity
   change.
5. Edit the milestone doc's relevant Phase section and Phase
   Checklist to reflect the new shape.
6. Update `<matrix-path>` if contract clauses, visible
   tests, hidden/generalization categories, adequacy checks, or
   mock policy changed.
7. `docs touch <milestone-path> <matrix-path>
   <project-path>decision-log.md`.
8. `docs index .` and `docs check . --stale 14`, then
   resume the corrected phase.

Plan revision is normal — every long-running milestone has at
least one. The audit trail is what makes it safe.
