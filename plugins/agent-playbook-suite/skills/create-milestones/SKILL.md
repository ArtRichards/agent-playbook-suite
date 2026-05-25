---
name: create-milestones
description: Create, advance, and complete milestones for a project whose foundation work is done. Drives the 10-phase TDD methodology (Define Contract, Write Tests RED, Create Fixtures, Run Tests RED, Update Interfaces, Implement Core, Update Wrappers, Run Tests GREEN, Integrate, Quality/Docs). Authors milestone + impl-log pairs via `docs new` and atomically archives them via `docs archive --cascade` on completion. Triggers on "create a milestone", "start M1", "next milestone", "begin implementation", "advance the project". Use after `project-foundation` has set up the docs tree.
---

# create-milestones

Drive milestone-level TDD work on a project whose foundation
artifacts already exist as docs-managed Markdown files. Each
milestone is a pair of docs (`<slug>.md` + `<slug>-impl.md`)
progressing through ten TDD phases, archived together with
`docs archive --cascade` on completion.

Requires [`docs-cli`](https://github.com/ArtRichards/docs-cli)
**v1.2.0 (M7+)** for the `Lifecycle:` field rename and
**v1.3.0 (M8+)** for `docs new --body-from -|<path>`. Ask the
user to upgrade if the host's `docs` binary is older.

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

**Read both before driving any milestone.**

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
4. **`Related: pairs-with`** is what bidirectionally links the
   milestone doc to its impl log — it's what makes
   `docs archive --cascade` find both at completion.
5. **One phase at a time.** Drive a single phase per exchange
   with the user; never batch. After each phase: append to the
   impl log, tick the checklist, `docs touch`, `docs check`,
   confirm before proceeding.
6. **`docs check <root> --stale 14`** runs at every phase
   boundary. Exit 2 blocks progression; exit 1 reviewed;
   exit 0 passes.
7. **Milestone completion uses `docs archive <slug>.md --cascade
   --reason "<reason>"`** — one atomic call that lands both
   task plan and impl log under `archive/<today>/` with
   `Lifecycle: archived` and a regenerated INDEX. Never
   hand-move files into `archive/` or hand-flip `Lifecycle:` to
   `archived`.

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

## Project context: CLAUDE.md

If the repo root has a `CLAUDE.md`, read it before driving any
phase. It documents the project's commit conventions, build/
test/quality commands, branch conventions, and any
project-specific rules — all things the phase work needs to
respect. The `project-foundation` skill scaffolds or extends
CLAUDE.md at its Phase 8; if it's missing, suggest the user
run `project-foundation`'s Phase 8 step (or scaffold from its
template) before starting milestone work.

## Per-phase output format

1. State phase number, name, and objective.
2. Briefly explain what will be done.
3. Ask clarifying questions if the milestone doc's phase
   section is sparse.
4. Do the work — write code, run tests.
5. Append a phase section to the impl log via body edit.
6. Tick the milestone doc's checklist; flip the impl log's
   progress-table cell.
7. `docs touch <slug>.md <slug>-impl.md status.md`.
8. `docs index <root>`.
9. `docs check <root> --stale 14`.
10. Confirm with the user before proceeding to the next phase.

## Completion checklist

When Phase 10 is done:

- [ ] Milestone-completion summary appended to both
      `<slug>.md` and `<slug>-impl.md`.
- [ ] Full quality gate green: project commands + `docs check`.
- [ ] `docs archive <slug>.md --reason "Milestone <M<N>>
      complete" --cascade` — accept the cascade prompt for the
      impl log; decline for long-lived parents like
      `milestone-plan.md` and `charter.md`.
- [ ] `status.md` updated: "Current milestone" advances; the
      milestone-progress row marked complete with the archive
      date.
- [ ] `docs touch status.md`.
- [ ] User asked whether to proceed to the next milestone.
