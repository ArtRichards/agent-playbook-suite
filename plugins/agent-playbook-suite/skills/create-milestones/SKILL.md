---
name: create-milestones
description: Create, advance, and complete milestones for a project whose foundation work is done. Drives the risk-aware, adequacy-checked 10-phase TDD methodology (Define Contract, Write Tests RED, Create Data/Fixtures, Run Tests RED Baseline, Update Base Interfaces, Implement Offline/Core Path, Update Tool/Wrapper Layer, Run Tests GREEN, Integrate/Accept/Dogfood, Quality/Docs). Authors milestone, impl-log, and test-matrix docs via `docs new` and atomically archives them via `docs archive <slug>.md --cascade-only '<slug>*'` on completion. Triggers on "create a milestone", "start M1", "next milestone", "begin implementation", "advance the project". Use after `project-foundation` has set up the docs tree.
---

# create-milestones

Drive milestone-level TDD work on a project whose foundation
artifacts already exist as docs-managed Markdown files. Each
milestone is a task plan, implementation log, and test-matrix
companion progressing through ten TDD phases, archived together
with `docs archive <slug>.md --cascade-only '<slug>*'` on completion.

Requires [`docs-cli`](https://github.com/ArtRichards/docs-cli) — always
run the newest release. The workflow relies on the
`Lifecycle:` metadata convention, `docs new --body-from -|<path>`,
and atomic multi-file `docs touch <file>...`.

## When this applies

- The user wants to create a new milestone for an existing project.
- The user wants to resume work on an in-flight milestone.
- The user wants to complete and archive a milestone.
- The user wants to track features or bug fixes within a milestone.

Do **not** apply when:

- The project has no docs tree yet, or the Definition of Ready is
  still `draft` — redirect to the `project-foundation` skill.
- The user wants a one-off TDD phase on something unrelated to a
  milestone — walk the phases by hand without invoking this skill.

## Substance lives in references/

This SKILL.md is intentionally short. The full procedure lives in:

- [`references/milestone-playbook.md`](references/milestone-playbook.md)
  — bootstrap, create, advance, complete, archive (with a worked
  example).
- [`references/tdd-phases.md`](references/tdd-phases.md) — the
  10-phase reference with per-phase docs-CLI touchpoints.
- [`../_shared/references/agentic-quality-model.md`](../_shared/references/agentic-quality-model.md)
  — risk levels, solution uncertainty, visible/hidden test layers,
  adequacy checks, hidden-test policy, and mock policy.

**Read the milestone playbook, phase guide, and shared quality
model before driving any milestone.**

## Invariants

1. **Verify the foundation first.** `docs list --root <root>
   --project <p> --role reference --lifecycle active` must
   include `definition-of-ready.md`. If it doesn't, redirect to
   `project-foundation` — don't bootstrap a docs root here.
2. **Every doc is *created* with `docs new`.** Never hand-author
   the metadata block. After creation, only two metadata fields
   are touched in place: `Lifecycle:` (when the milestone
   advances draft → active → archived) and `Related:` (for typed
   edges between the milestone doc and its impl log). Every
   other field is owned by the CLI; use `docs touch` to bump
   `Updated:`.
3. **Controlled-vocab field is `Lifecycle:`** (M7+). A
   controlled-vocab `Status:` line is wrong; `docs check`
   exits 2.
4. **`Related:` edges link the milestone artifacts.** The
   milestone doc carries `pairs-with` links to its impl log and
   test matrix. Optional `parent-of` links can express hierarchy,
   but candidate discovery follows `pairs-with` and `child-of`
   only — one hop, no transitive cascade — so do not rely on
   `parent-of` alone. The milestone doc's own
   `child-of: milestone-plan.md` edge stays as it is. What to
   avoid is the reverse direction: a document that
   **deliberately outlives** the milestone must not declare
   `child-of` the milestone doc, because at docs 2.0 a
   still-active document declaring `child-of` a document the
   archive plan would move makes `docs archive` refuse at exit 2,
   naming both ends and writing nothing. Make such a document a
   **pair** and give it a slug the completion glob does not
   match.
5. **One phase at a time.** Drive a single phase per exchange
   with the user; never batch. After each phase: append to the
   impl log, tick the checklist, `docs touch`, `docs check`,
   confirm before proceeding.
6. **Risk, adequacy, and taste are first-class.** Every milestone
   doc records `Risk Level`, a behavior `Contract` (including
   taste anchors per the shared quality model's Taste model),
   `Test Strategy For This Milestone`, and a linked `Test
   Matrix`. Use the shared quality model to decide Lite /
   Standard / High gates. Taste findings raised in review are
   must-triage — fixed or waived with a logged reason, never
   dropped. In interactive runs there is no conductor, so taste
   waivers fall to the operator at the phase boundary.
7. **High-risk RED checkpoint.** For High-risk milestones,
   stop after Phase 4's RED baseline and ask for operator
   approval before implementation continues, unless a project
   policy explicitly allows automatic continuation.
8. **`docs check <root> --stale 14`** runs at every phase
   boundary. Exit 2 blocks progression; exit 1 reviewed;
   exit 0 passes.
9. **Milestone completion is preview, then explicit scope.**
   Bare `--cascade` is **retired at docs 2.0** — it refuses at
   exit 2 and writes nothing. Preview the neighbourhood, then
   write exactly the scope you meant:

   ```sh
   docs archive <slug>.md --cascade-dry-run --cascade-only '<slug>*'
   docs archive <slug>.md --cascade-only '<slug>*' --reason "<reason>"
   ```

   **Preview the plan you are about to write** — pass the same
   `--cascade-only` glob to the dry run. A bare `--cascade-dry-run`
   lists the candidate neighbourhood but selects nothing, so it does
   not rehearse the scoped write.

   `'<slug>*'` reaches the milestone's own companions — impl log and
   test matrix — and cannot reach `milestone-plan.md` or `charter.md`,
   which is the job the interactive prompt used to do. It is **not** a
   guarantee that those are the only matches: a slashless glob matches
   the basename at any depth, so a `<slug>-release-log.md` or a
   `quality/<slug>-quality-log.md` matches too. Read the preview and
   narrow the glob if it selected something you meant to keep active.
   The write is one atomic call landing the set under
   `archive/<today>/` with `Lifecycle: archived` and a regenerated
   INDEX. Never hand-move files into `archive/` or hand-flip
   `Lifecycle:` to `archived`.
10. **Explore only clear solution uncertainty.** Invoke the companion
    `explore` skill automatically when the shared quality model's
    high-threshold gates show that implementation is clearly uncertain or the
    problem is genuinely novel, including recovery from a concretely
    invalidated route. Do not invoke it for routine work or agent unfamiliarity.
    Keep its full registry in the implementation log; record only the
    disposition and a link in the milestone's Decisions section.

## Bootstrap requirements (what must exist)

Before this skill does anything, the project's docs root must
have:

- `.docs.toml` at the root with `[project] name = "<slug>"`.
- A green Definition of Ready: `definition-of-ready.md` with
  `Lifecycle: active` (or `done`), `Role: reference`.
- A milestone plan: `milestone-plan.md` with `Role: plan`,
  `Lifecycle: active`, listing M1..Mn.
- A status doc: `status.md` with `Role: status`,
  `Lifecycle: active`.
- `docs check <root> --stale 14` exits 0 or 1 (not 2).

If any of these is missing, redirect to `project-foundation`.

## Project context: CLAUDE.md, AGENTS.md, or equivalent

Before driving any phase, read the project-root agent context that
exists for the host: `CLAUDE.md`, `AGENTS.md`, or an equivalent
project instruction file. It documents the project's commit
conventions, build/test/quality commands, branch conventions, and
any project-specific rules — all things the phase work needs to
respect. The `project-foundation` skill scaffolds or extends
`CLAUDE.md` and/or `AGENTS.md` at its Phase 8; if no agent context
exists, suggest the user run `project-foundation`'s Phase 8 step
(or scaffold from its template) before starting milestone work.

## Per-phase output format

1. State phase number, name, and objective.
2. Briefly explain what will be done.
3. Ask clarifying questions if the milestone doc's phase
   section is sparse.
4. Do the work — write code, run tests.
5. Append a phase section to the impl log via body edit.
6. Record risk-gate, adequacy, hidden/generalization, and mock
   evidence selected for the phase.
7. Tick the milestone doc's checklist; flip the impl log's
   progress-table cell.
8. `docs touch <slug>.md <slug>-impl.md <slug>-test-matrix.md
   status.md`.
9. `docs index <root>`.
10. `docs check <root> --stale 14`.
11. Confirm with the user before proceeding to the next phase.

## Completion checklist

When Phase 10 is done:

- [ ] Milestone-completion summary appended to both
      `<slug>.md` and `<slug>-impl.md`.
- [ ] Test matrix updated with visible tests/checks, selected
      hidden/generalization, property/stateful, mutation,
      fuzz/benchmark/security/schema, and mock-audit status.
- [ ] Adequacy results summarized, including hidden-generalization
      gap when available and explicit follow-ups for skipped deep
      gates.
- [ ] Taste findings triaged: each fixed or waived with a logged
      reason (High-risk waivers operator-approved); recurring
      findings/waivers noted for promotion into CLAUDE.md /
      AGENTS.md conventions or future taste anchors.
- [ ] Boundary sweep done: speculative outputs this milestone
      leaves behind are ledgered in `followup-log.md` (with named
      consumer) or removed; inbound ledger entries naming this
      milestone as consumer are closed or challenged.
- [ ] Selected quality gate green: configured project commands +
      `docs check`.
- [ ] `docs archive <slug>.md --cascade-dry-run --cascade-only
      '<slug>*'` to preview the exact plan, then the same command
      with `--reason "Milestone <M<N>> complete"` and without
      `--cascade-dry-run` to write it. The glob takes the impl log
      and test matrix and cannot reach `milestone-plan.md` or
      `charter.md`; check the preview for anything else it matched.
      There is no prompt to answer at docs 2.0.
- [ ] `status.md` updated: "Current milestone" advances; the
      milestone-progress row marked complete with the archive
      date.
- [ ] `docs touch status.md`.
- [ ] User asked whether to proceed to the next milestone.
