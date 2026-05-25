# TDD Phases — 10-Phase Reference

The canonical phase definitions for milestone-driven TDD work.
The orchestration that drives them lives in
[`milestone-playbook.md`](milestone-playbook.md).

Each phase below lists: objective, activities, deliverables, exit
criteria, and the docs-CLI touchpoints (every phase ends with at
least a `docs touch` on the impl log).

## Phase 1 — Define Contract

- **Objective:** Specify the interface — schemas, function
  signatures, type hints, docstrings. No business logic yet.
- **Activities:** Define models/enums; document parameters and
  return types; enumerate use cases.
- **Deliverables:** Contract file(s) created; docstrings with
  examples; no implementation.
- **Exit:** Types compile; no reliance on implementation; ready
  for tests.
- **Docs touchpoints:**
  - Append a Phase 1 section to `<slug>-impl.md` body.
  - Tick `[x] Phase 1` in `<slug>.md`'s checklist; flip the
    progress-table cell from `Pending` to `Complete`.
  - `docs touch <slug>.md <slug>-impl.md status.md`.
  - `docs check <root> --stale 14` exit 0 or 1.

## Phase 2 — Write Tests (RED)

- **Objective:** Express desired behaviour as failing tests
  before any implementation.
- **Activities:** Write happy-path and edge-case tests; include
  fixture stubs; consider pagination, filtering, error paths.
- **Deliverables:** Test module(s) with clear names and
  docstrings; minimum coverage targets noted.
- **Exit:** Tests import cleanly; expected to fail due to
  missing implementation.
- **Docs touchpoints:** standard per-phase set above.

## Phase 3 — Create Data/Fixtures

- **Objective:** Provide synthetic data/builders to exercise the
  tests.
- **Activities:** Add diverse fixtures; cover all enums /
  statuses / variants; ensure correctness for money,
  date-handling, edge sizes.
- **Deliverables:** Fixture files or factory helpers; integrity
  checked.
- **Exit:** Data loads cleanly; represents every required variant.
- **Docs touchpoints:** standard per-phase set above.

## Phase 4 — Run Tests (RED Baseline)

- **Objective:** Confirm tests fail for the right reasons.
- **Activities:** Run the test suite; capture failure summary
  verbatim.
- **Deliverables:** Test output with failure reasons noted in
  the impl log.
- **Exit:** Failing tests trace to missing implementation, not
  misconfiguration.
- **Docs touchpoints:** standard per-phase set above. Paste the
  RED baseline output into the Phase 4 log section verbatim.

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
  path(s).
- **Activities:** Run focused suites; iterate fixes; keep a
  changelog in the impl log.
- **Deliverables:** Passing test output; notes on any flaky cases.
- **Exit:** All targeted tests green; quality checks (format,
  lint, typecheck) clean.
- **Docs touchpoints:** standard per-phase set above. Paste
  GREEN test output verbatim into the Phase 8 log section. If
  anything is RED, STOP — fix the root cause; do not relax a
  test.

## Phase 9 — Implement Online/Integration

- **Objective:** Add the online / remote / integration path.
- **Activities:** Add HTTP/OAuth/retry behaviour; reuse
  decorators/utilities; mirror offline validation.
- **Deliverables:** Online provider/path; integration tests or
  stubs; retry/backoff configuration.
- **Exit:** Integration tests prepared or passing; clear TODOs
  for any blocked items.
- **Docs touchpoints:** standard per-phase set above. Many
  milestones have no online surface; in that case repurpose
  Phase 9 for dogfooding against realistic fixtures and
  document the pass criteria + measured metrics in the Phase 9
  log section.

## Phase 10 — Quality, Docs, Refactor

- **Objective:** Final polish and handoff readiness.
- **Activities:**
  - Run the full quality gate (`make format && make lint &&
    make typecheck && make test` or project equivalent).
  - Refactor for clarity (the `simplify` skill may help here).
  - Update user-facing docs.
  - Append a milestone-completion summary to both the milestone
    doc and the impl log.
- **Deliverables:** Updated documentation set; implementation
  checklist completed; open questions noted; release notes if
  applicable.
- **Exit:** Quality gate green; documentation current; handoff
  notes written. Ready for `docs archive <slug>.md --cascade
  --reason "<reason>"`.
- **Docs touchpoints:**
  - Append "Milestone-completion summary" sections to both
    `<slug>.md` and `<slug>-impl.md`.
  - `docs touch` both.
  - `docs check <root> --stale 14` must exit 0 or 1.
  - **Do not archive yet** — the playbook's Step 4 owns the
    archive call so the project-level status update happens in
    one place.

## Per-phase docs hygiene checklist

After every phase, regardless of which one:

- [ ] Phase section appended to `<slug>-impl.md` body (objective,
      files, actions, results, decisions).
- [ ] Progress-table cell in `<slug>-impl.md` flipped to
      `Complete`.
- [ ] `[x]` ticked in `<slug>.md`'s Phase Checklist.
- [ ] `status.md`'s "Current Phase" line updated.
- [ ] `docs touch <slug>.md <slug>-impl.md status.md`.
- [ ] `docs index <root>` regenerated.
- [ ] `docs check <root> --stale 14` exit 0 or 1.
- [ ] User confirmation before starting the next phase.

## Phase ordering invariants

- **Phases run sequentially.** Never skip ahead.
- **Phase 4 must reveal RED for the right reasons** — a Phase 4
  that already shows GREEN means Phase 2 didn't test what was
  intended; go back.
- **Phase 8 must be fully GREEN before Phase 9.** A flaky test
  is not GREEN; either fix the flake or scope it out with a
  documented TODO before proceeding.
- **Phase 10 always runs**, even for milestones that feel small.
  The completion summary + docs touch is what makes archive
  meaningful.

## When a phase reveals the plan was wrong

If during a phase you discover the milestone plan needs
substantial revision:

1. Stop the phase. Do not press on with the wrong plan.
2. Append a decision-log entry capturing what was discovered and
   what needs to change.
3. Edit the milestone doc's relevant Phase section and Phase
   Checklist to reflect the new shape.
4. `docs touch <slug>.md` and the decision log.
5. Resume the (now corrected) phase.

Plan revision is normal — every long-running milestone has at
least one. The audit trail is what makes it safe.
