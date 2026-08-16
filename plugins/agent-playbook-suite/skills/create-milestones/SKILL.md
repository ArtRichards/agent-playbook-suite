---
name: create-milestones
description: Create, advance, and complete semantic milestones for a project whose foundation work is done. Selects and claims work from the canonical milestone-plan.md tracker, drives the risk-aware 10-phase TDD methodology (Define Contract, Write Tests RED, Create Fixtures, Run Tests RED, Update Interfaces, Implement Core, Update Wrappers, Run Tests GREEN, Integrate, Quality/Docs), authors milestone, impl-log, and test-matrix docs via `docs new`, and archives their explicit scope on completion. Triggers on "create a milestone", "start session-storage", "next milestone", "begin implementation", "advance the project". Use after `project-foundation` has set up the docs tree.
---

# create-milestones

Drive milestone-level TDD work on a project whose foundation
artifacts already exist as docs-managed Markdown files. Each
milestone is a task plan, implementation log, and test-matrix
companion progressing through ten TDD phases. Semantic identity,
order, execution state, dependencies, and derived next work live in
`milestone-plan.md`; completion archives the explicitly previewed
artifact scope with
`docs archive <milestone-path> --cascade-only '<archive-scope>'`.

Requires [`docs-cli`](https://github.com/ArtRichards/docs-cli) 2.0 or newer —
prefer the newest release. The workflow relies on the
`Lifecycle:` metadata convention, `docs new --body-from -|<path>`,
atomic multi-file `docs touch <file>...`, reciprocal `docs relate`,
and safe explicit archive selection.

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
- [`../_shared/references/milestone-tracker.md`](../_shared/references/milestone-tracker.md)
  — the canonical tracker schema, semantic identity, eligibility,
  claim, state-transition, dependency, and relationship rules.
- [`../_shared/references/operator-interaction.md`](../_shared/references/operator-interaction.md)
  — shared requirements for grounded, understandable operator
  questions, approvals, and decisions.

**Read the milestone playbook, phase guide, shared quality model,
milestone-tracker contract, and operator-interaction policy before
driving any milestone.**

The milestone-tracker contract is required. Before selecting, creating,
advancing, or completing a milestone, verify that the shared reference exists
and read it in full. If it is absent, stop and explain that the full Agent
Playbook Suite installation is incomplete; recommend reinstalling or updating
the complete suite. Never reconstruct the contract from the playbook, project
artifacts, or another skill.

## Resolve project paths once

After locating the docs root, resolve these root-relative values and use them
for every docs-cli path operand. Run the displayed root-relative commands from
that docs root; `--root <root>` selects configuration but does not rebase every
positional file operand from another working directory:

- `<project-path>` is empty when this project's docs are flat in a dedicated
  root; in a shared root it is the directory containing this project's
  tracker, with a trailing slash (for example `specs/payments/`).
- `<artifact-stem>` is the physical basename without `.md`. For new work it is
  `<slug>` in a dedicated root and `<project>-<slug>` in a shared root. For an
  already materialized or historical milestone, recover it from the
  established tracker link and preserve it even when it predates this
  convention.
- `<milestone-path>`, `<impl-path>`, and `<matrix-path>` are respectively
  `<project-path><artifact-stem>.md`,
  `<project-path><artifact-stem>-impl.md`, and
  `<project-path><artifact-stem>-test-matrix.md`.
- `<tracker-path>` and `<status-path>` are
  `<project-path>milestone-plan.md` and `<project-path>status.md`.
- `<archive-scope>` is the root-relative glob
  `<project-path><artifact-stem>-*`, passed as one shell-quoted argument. With
  an empty project path and a new milestone this is the familiar
  `'<slug>-*'`.

The semantic `<slug>` remains the tracker identity and branch component;
`<artifact-stem>` is only physical identity. Before activation, validate that
the stem and all three derived basenames are globally unambiguous in the docs
root. Same-directory Markdown links may remain relative, such as
`[<slug>](<artifact-stem>.md)`. Never fall back to a bare filename merely
because it works for a dedicated root, and never mass-rename established
activated or historical paths.

## Invariants

1. **Verify the foundation first.** `docs list --root <root>
   --project <p> --role reference --lifecycle active` must
   include `<project-path>definition-of-ready.md`. If it doesn't, redirect to
   `project-foundation` — don't bootstrap a docs root here.
2. **The tracker owns milestone scheduling.** Its exact
   `Order | Milestone | State | Depends on | Notes` table owns
   semantic identity, order, durable state, dependencies, and derived
   next work. `<status-path>` is a narrative summary, never a second
   milestone table or scheduler. For new work, select one eligible
   `planned` row, immediately re-read the table, change only that row
   to `active`, then `docs touch <tracker-path>` and run
   `docs check` before creating artifacts, delegating production work,
   or creating a branch. Never silently reclaim an `active` row.
3. **Every doc is *created* with `docs new`.** Never hand-author
   the metadata block. After creation, direct **metadata** edits are limited to
   `Lifecycle:` when a draft artifact becomes active and free-form
   `Related:` links such as companion `pairs-with` edges. Body content remains
   editable through the documented workflow. Use
   `docs relate` for every recognized reciprocal pair, `docs archive`
   to write archived lifecycle metadata, and `docs touch` to bump
   `Updated:` after body or permitted metadata edits. Every other
   managed field is CLI-owned.
4. **Controlled-vocab field is `Lifecycle:`.** A
   controlled-vocab `Status:` line is wrong; `docs check`
   exits 2.
5. **`Related:` edges link the milestone artifacts.** The
   milestone doc carries `pairs-with` links to its impl log and
   test matrix. The milestone doc's own
   `child-of: <tracker-path>` edge stays as it is. What to
   avoid is the reverse direction: a document that
   **deliberately outlives** the milestone must not declare
   `child-of` the milestone doc, because at docs 2.0 a
   still-active document declaring `child-of` a document the
   archive plan would move makes `docs archive` refuse at exit 2,
   naming both ends and writing nothing. Make such a document a
   **pair** and give it a slug the completion glob does not
   match. Synchronize tracker-derived `precedes`/`follows` and
   `depends-on`/`required-by` pairs with `docs relate`, including
   its required `--reason` when an endpoint is archived. Sequence,
   dependency, blocker, `pairs-with`, and `child-of` relationships
   provide navigation or archive candidates; none authorizes an
   archive write.
6. **One phase at a time.** Drive a single phase per exchange
   with the user; never batch. After each phase: append to the
   impl log, tick the checklist, `docs touch`, `docs check`,
   confirm before proceeding.
7. **Risk, adequacy, and taste are first-class.** Every milestone
   doc records `Risk Level`, a behavior `Contract` (including
   taste anchors per the shared quality model's Taste model),
   `Test Strategy For This Milestone`, and a linked `Test
   Matrix`. Use the shared quality model to decide Lite /
   Standard / High gates. Taste findings raised in review are
   must-triage — fixed or waived with a logged reason, never
   dropped. In interactive runs there is no conductor, so taste
   waivers fall to the operator at the phase boundary.
8. **High-risk RED checkpoint.** For High-risk milestones,
   stop after Phase 4's RED baseline and ask for operator
   approval before implementation continues, unless a project
   policy explicitly allows automatic continuation.
9. **`docs check . --stale 14`** runs from the docs root at every phase
   boundary. Exit 2 blocks progression; exit 1 reviewed;
   exit 0 passes.
10. **Milestone completion is preview, then explicit scope.**
   Bare `--cascade` is **retired at docs 2.0** — it refuses at
   exit 2 and writes nothing. Preview the neighbourhood, then
   write exactly the scope you meant:

   ```sh
   docs archive <milestone-path> --cascade-dry-run --cascade-only '<archive-scope>'
   docs archive <milestone-path> --cascade-only '<archive-scope>' --reason "<reason>"
   ```

   **Preview the plan you are about to write** — pass the same
   `--cascade-only` glob to the dry run. A bare `--cascade-dry-run`
   lists the candidate neighbourhood but selects nothing, so it does
   not rehearse the scoped write.

   `<archive-scope>` selects the milestone's impl log and test matrix from
   its one-hop candidates and, in a shared root, anchors selection to this
   project directory. It is **not** proof that those are the only matches:
   `<project-path><artifact-stem>-release-log.md` would match too. Require the
   preview to show the primary plus exactly the intended companions;
   stop and narrow the scope or repair the candidate relationships if
   it selects anything else. Also stop on any occupied archive destination;
   never falsify the archive date or rename a frozen identity to bypass a
   same-basename collision. The operation validates its complete plan
   before moving the set under `<archive-dir>/<today>/`, writes
   `Lifecycle: archived`, rebases metadata and body links, and
   regenerates INDEX. Never hand-move files into `archive/` or
   hand-flip `Lifecycle:` to `archived`.
11. **Resume completion from evidence, never by replaying archive.** Before
    preview/apply, classify the three derived paths and tracker row. If all
    three artifacts are already archived together, the tracker row is still
    `active`, and its link was rebased, verify the archive witnesses and docs
    state, then resume at tracker/status completion without running archive
    again or editing archived files. If the same verified set already has a
    `complete` row, run the final docs verification and report completion.
    Any mixed live/archived set, missing or extra milestone companion,
    conflicting date, stale tracker link, or other unexplained state stops for
    reconciliation.
12. **Explore only clear solution uncertainty.** Invoke the companion
    `explore` skill automatically when the shared quality model's
    high-threshold gates show that implementation is clearly uncertain or the
    problem is genuinely novel, including recovery from a concretely
    invalidated route. Do not invoke it for routine work or agent unfamiliarity.
    Keep its full registry in the implementation log; record only the
    disposition and a link in the milestone's Decisions section.

## Bootstrap requirements (what must exist)

Before this skill does anything, the project's docs root must
have:

- `.docs.toml` at the root. Resolve the owning `Project: <p>` from the
  project's tracker and foundation docs; a dedicated root normally uses the
  same default under `[project]`, while a shared root can contain multiple
  explicit project values.
- A green Definition of Ready: `<project-path>definition-of-ready.md` with
  `Lifecycle: active` (or `done`), `Role: reference`.
- A milestone plan: `<tracker-path>` with `Role: plan`,
  `Lifecycle: active`, and the canonical five-column tracker table.
- A status doc: `<status-path>` with `Role: status`,
  `Lifecycle: active`.
- `docs check . --stale 14` exits 0 or 1 (not 2).

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
3. Inspect the milestone and project evidence first. If its phase
   section still needs clarification, ask only the unresolved
   questions and follow the shared operator-interaction policy.
4. Do the work — write code, run tests.
5. Append a phase section to the impl log via body edit.
6. Record risk-gate, adequacy, hidden/generalization, and mock
   evidence selected for the phase.
7. Tick the milestone doc's checklist; flip the impl log's
   progress-table cell.
8. `docs touch <milestone-path> <impl-path> <matrix-path>
   <status-path>`.
9. `docs index .`.
10. `docs check . --stale 14`.
11. Confirm with the user before proceeding to the next phase.

## Completion checklist

When Phase 10 is done:

- [ ] Milestone-completion summary appended to both
      `<milestone-path>` and `<impl-path>`.
- [ ] `<matrix-path>` updated with visible tests/checks, selected
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
      leaves behind are ledgered in `<project-path>followup-log.md` (with named
      consumer) or removed; inbound ledger entries naming this
      milestone as consumer are closed or challenged.
- [ ] Selected quality gate green: configured project commands +
      `docs check`.
- [ ] Tracker row is `active`, its semantic slug matches all three
      artifact paths derive from one validated `<artifact-stem>`, and its
      `Milestone` cell maps the semantic slug to the milestone doc (normally
      the relative `<artifact-stem>.md`); stored dependencies and recognized
      reciprocal relationships match the tracker contract.
- [ ] Completion checkpoint classified before any archive command. A normal
      live set proceeds; a verified already-archived set follows the
      fail-closed resume rules without replaying archive or touching archived
      files; mixed or unexplained state stops.
- [ ] `docs archive <milestone-path> --cascade-dry-run --cascade-only
      '<archive-scope>'` to preview the exact plan, then the same command
      with `--reason "Milestone <slug> complete"` and without
      `--cascade-dry-run` to write it. The glob takes the impl log
      and test matrix from the one-hop candidates; check that the
      preview selects no other document. Relationships are candidates,
      not permission. There is no prompt to answer at docs 2.0.
- [ ] Post-archive `docs check` passes and the tracker link has been
      rebased to the dated archive path.
- [ ] Tracker row changed `active` → `complete` only after that
      archive/check evidence, preserving `Order`, semantic slug,
      dependencies, and rebased link.
- [ ] `<status-path>` updated as a completion/current-activity narrative that
      links to the tracker without duplicating rows or storing a next-work
      choice.
- [ ] `docs touch <tracker-path> <status-path>`, `docs index .`,
      and final `docs check . --stale 14` passed.
- [ ] User response names the derived next eligible milestone (or why none is
      eligible) and asks whether to proceed.
