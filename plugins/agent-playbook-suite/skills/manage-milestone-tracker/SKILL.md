---
name: manage-milestone-tracker
description: Manage semantic milestone identity, order, execution state, dependencies, blockers, and deterministic next-work selection in a docs-backed milestone-plan.md tracker. Use when the operator asks to inspect, initialize or migrate, insert, reorder or normalize, rename planned work, claim, pause, resume, cancel, complete, change dependencies or blockers, validate, or determine the next milestone. Does not create full milestone artifact sets, run TDD phases, archive milestones, or commit code.
---

# manage-milestone-tracker

Manage the canonical milestone tracker without coupling execution order to
filenames. Keep this skill thin: the schema and algorithms live in one shared
contract, and docs-cli owns metadata, relationships, moves, and validation.

## Read first

- Read
  [`../_shared/references/milestone-tracker.md`](../_shared/references/milestone-tracker.md)
  in full before inspecting or changing a tracker.
- Read
  [`../_shared/references/operator-interaction.md`](../_shared/references/operator-interaction.md)
  before asking any clarification, confirmation, or decision.
- Read the project-root `AGENTS.md`, `CLAUDE.md`, or equivalent and the
  project's qualified tracker, status summary, Definition of Ready, and
  relevant milestone artifacts.
- Use the bundled `docs` skill and docs-cli 2.0 or newer for `docs new`,
  `docs mv`, `docs relate`, `docs touch`, `docs index`, and `docs check`.

The milestone-tracker contract is required. Verify that the shared reference
exists before inspecting or changing a tracker. If it is absent, stop and
explain that the full Agent Playbook Suite installation is incomplete;
recommend reinstalling or updating the complete suite. Never reconstruct the
contract from project artifacts, another skill, or remembered conventions.

## Boundaries

This skill owns direct tracker operations: inspect, initialize or migrate,
insert, reorder or normalize, planned-only rename, claim, pause, resume,
cancel, complete, dependency or blocker changes, derived-next reporting, and
tracker validation.

It does not:

- create a milestone's task plan, implementation log, or test matrix;
- execute TDD phases or implementation;
- archive milestone artifacts;
- commit or push;
- create mandatory stubs, a tracker utility, lock service, or scheduler; or
- rename an identity after activation.

Use `project-foundation` for normal tracker initialization,
`create-milestones` to materialize and interactively execute a selected row,
`ship-milestone` for autonomous execution, and `sync-and-commit` for final
consistency and git closure.

## Procedure

1. **Locate scope.** Find the `.docs.toml` root, resolve the owning `Project:`,
   and establish the contract's single root-relative `<project-path>` (empty
   for a dedicated root). Find `<project-path>milestone-plan.md` and
   `<project-path>status.md`; in a shared root, never combine rows or paths
   from different projects. Resolve each row's physical `<artifact-stem>` from
   its existing link/path or, for new work, from the contract's dedicated- vs
   shared-root rule. Change the working directory to the docs root before any
   docs-cli operation; `--root` does not rebase relative document operands.
2. **Inspect before changing.** Read the tracker, status summary, DoR, relevant
   docs and relationships, and `docs check` output. For migration or uncertain
   state, also inspect logs, branches, lifecycle, and archives.
3. **Validate the current model.** Apply the shared contract's narrow
   validation set. Report objective defects before attempting an unrelated
   mutation.
4. **Derive the operation.** Show the affected rows, physical artifact stems,
   state/order changes, and recognized relationship additions/removals. Check
   the root-global physical basename set. Ask only when evidence cannot
   resolve an ambiguity or the requested change alters operator-owned intent;
   use the shared explanation policy. Migration is always proposal-first.
5. **Re-read immediately before writing.** Compare the canonical table with
   the inspected snapshot. If it changed, write nothing and return the conflict
   for reconciliation.
6. **Apply one coherent mutation.** For normal operations, edit only tracker
   body content. Initialization and legacy migration also own the bounded
   status-body edit required by the contract: keep its narrative, link the
   tracker, and remove independent schedule/next claims. Use `docs mv` for a
   permitted planned rename and `docs relate add/remove` for recognized
   relationship pairs; never hand-edit CLI-owned metadata or one inverse.
7. **Synchronize and prove.** Qualify every docs-cli document operand with
   `<project-path>`. Run `docs touch <project-path>milestone-plan.md` plus any
   changed body docs; initialization and migration must touch the qualified
   tracker and status paths together. Then run `docs index .` and
   `docs check .`. Re-read the table and report the resulting rows,
   relationships, and derived next work.

Do not create an extra confirmation gate for a clear, reversible request. A
user who says "move export-reporting before audit-log" has already authorized
that reorder; preview it, preserve identities, apply it, and validate. Stop for
operator input only when the target or intended ordering is genuinely
ambiguous.

## Operations

### Inspect, validate, and next

Inspect and validate are read-only. For `next`, derive eligibility and report
the lexicographically first slug in the lowest eligible order cohort. Name
unmet dependencies, live blockers, or DoR failure when nothing is eligible.
Do not claim unless the operator or a consuming workflow asked to start/claim.

### Initialize or migrate

If no plan exists and foundation context is otherwise present, create it with
`docs new plan <project-path>milestone-plan` and the exact five-column table.
It begins with `Lifecycle: draft`; after the table, project scope, Definition
of Ready, and initial dependencies validate, promote only that field to
`Lifecycle: active`. Update `<project-path>status.md` to retain the project
narrative, add `[Milestone Plan](milestone-plan.md)`, and remove independent
schedule or next-work claims. If status is absent, create it with
`docs new status <project-path>status` and promote it to `Lifecycle: active`.
Then run the contract's atomic qualified
tracker/status touch before index/check. Never leave a newly initialized
tracker draft when returning it as ready for delivery.

For a legacy plan, build the complete proposal from all evidence and get
confirmation for each ambiguous mapping before writing. The confirmed
migration also normalizes the qualified status body; if it is absent, include
`docs new status <project-path>status` and its promotion to
`Lifecycle: active` in the proposal. Touch both qualified paths before
index/check. Preserve existing activated and historical slugs; do not
mass-rename their linked physical artifact stems or generate milestone stubs.
Record any frozen legacy flattened-basename collision as an explicit delivery
blocker for operator resolution.

### Insert, reorder, and normalize

Insert a meaningful project-unique slug at an available integer order and
validate its would-be physical artifact stem root-globally; equal orders form
a parallel cohort. Reorder by changing only order values.
Normalize distinct cohorts to 100-point gaps when useful, retaining equal
cohorts and every identity. After any order change, regenerate only the
materialized adjacent-cohort `precedes`/`follows` pairs.

### Claim, pause, resume, cancel, and complete

- Claim exactly one eligible `planned` row by re-reading and changing it to
  `active` before production delegation or branch creation.
- Pause only an `active` row and require a reason in Notes.
- Resume `paused` to `active`; never return it to `planned`.
- Cancel `planned`, `active`, or `paused` work and require a reason.
- Mark `active` work `complete` only after the owning delivery workflow proves
  implementation, review, simplify, archive, and final docs consistency.

Never silently reclaim or expire `active` work.

### Rename planned work

Rename only a `planned` identity. Reject reserved suffixes and all milestone or
companion collisions, including flattened archive basenames. Update tracker and
dependency cells, recompute the dedicated- or shared-root artifact stem, move
any intentional stub or pre-created companions with `<project-path>`-qualified
`docs mv` operands, and resynchronize recognized relationships. Reject any
semantic-slug or artifact-stem rename after the row has ever been active.

### Dependencies and blockers

Keep durable milestone prerequisites in `Depends on`; when both milestone docs
exist, synchronize `depends-on`/`required-by` with `docs relate`. Keep transient
conditions in `blocked-by`/`blocks` and remove them when cleared. A blocker
requires a materialized milestone doc; do not invent a stored blocked state or
create a stub automatically.

Use `<project-path>`-qualified root-relative source and target operands for
every `docs relate` call. When an endpoint is archived, include the required
`--reason` audit text. Never derive dependency from order or use relationships
as archive membership.

## Return

Report:

- project and tracker path;
- operation applied, or read-only result;
- affected rows, physical artifact stems, and recognized relationship changes;
- derived next eligible milestone, or exact no-eligible reasons;
- `docs check` result; and
- any unresolved conflict requiring operator reconciliation.
