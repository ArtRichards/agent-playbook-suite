# Milestone Tracker Contract

Use this contract whenever a suite workflow reads or changes milestone order,
identity, execution state, dependencies, blockers, or next-work selection.

## Project scope and paths

Resolve one `<project-path>` before reading or writing. It is the
docs-root-relative directory prefix for this project's live docs, including a
trailing slash when nonempty: use the empty string for a dedicated project
root, or a path such as `specs/payments/` in a shared root. The canonical
tracker is `<project-path>milestone-plan.md`; the project summary is
`<project-path>status.md`.

Change the working directory to the docs root before any docs-cli operation.
`--root` selects a docs tree but does not rebase relative `FILE`, `SOURCE`, or
`TARGET` operands; do not use it as a substitute for this `cd`. Qualify every
docs-cli document operand and every `Related:` target with `<project-path>`;
`docs new` receives `<project-path><artifact-stem>` without `.md` for a
milestone artifact. Markdown body links between docs in the same project
directory remain relative: status uses `[Milestone Plan](milestone-plan.md)`,
while a tracker row uses `[session-storage](session-storage.md)` in a dedicated
root or `[session-storage](payments-session-storage.md)` in a shared
`payments` project.
Each project uses exactly one path for all live foundation and milestone docs.
This path boundary, together with `Project:`, lets a shared root safely host
the same semantic milestone slug for different projects while the docs are
live. The physical artifact-stem rule below also keeps their flattened archive
destinations distinct.

`status.md` is a narrative summary and link surface. It must link to the
canonical tracker without copying tracker rows or independently storing
schedule or next-work claims.

## Canonical table

`milestone-plan.md` contains exactly one canonical table with these five
columns, in this order. This dedicated-root example has identical semantic slug
and physical artifact stem:

```markdown
| Order | Milestone | State | Depends on | Notes |
|---:|---|---|---|---|
| 100 | auth-contracts | planned | — | Session and token contract |
| 200 | [session-storage](session-storage.md) | active | auth-contracts | — |
```

Rules:

- `Order` is an integer. Start distinct cohorts at `100`, `200`, `300`, and so
  on. Equal values mean the milestones may proceed in parallel. Three-digit
  spacing is a convention, not a ceiling.
- `Milestone` is the project-unique semantic slug. It may be plain text while
  planned and unmaterialized, or a Markdown link once a milestone document or
  intentional stub exists. Link text remains the semantic slug; the relative
  destination uses the physical artifact stem (`session-storage.md` in a
  dedicated root, or for example `payments-session-storage.md` in a shared
  root). Do not create a stub merely to make the cell a link.
- `State` is exactly one of `planned`, `active`, `paused`, `complete`, or
  `cancelled`.
- `Depends on` is `—` or a comma-separated list of semantic slugs that also
  occur in this project's table. It records durable prerequisite facts, not
  transient blocking conditions.
- `Notes` is concise exceptional context. It must contain a reason for
  `paused` and `cancelled` rows.

The plan may also contain milestone detail sections, sequencing explanation,
demo checkpoints, and buffer notes. Those sections define scope; the table is
the only authority for identity, order, stored execution state, dependencies,
and derived next work.

Do not add tracker columns for phase, branch, owner, claim time, readiness,
blocking, or next. Phase and branch live in the implementation log. `active`
is the durable claim. Readiness, blocking, and next are derived.

## Semantic identity

For new work, choose a meaningful lowercase kebab-case slug such as
`session-storage`, not an ordinal identity such as `m4` or `m4a`. The slug is
stable identity; changing `Order` never renames files, branches, or links.

- The `Project:` slug is unique across the repository, including separate docs
  roots. Before creating or adopting one, inspect every docs-managed tree and
  reject a value already owned by a different project path. This makes
  `<project>/` a repository-unique branch namespace rather than merely a docs
  metadata label.
- Milestone slugs are unique within the owning `Project:`. Shared docs roots
  may contain the same semantic milestone slug in different projects.
- Resolve one physical `<artifact-stem>` per row. In a dedicated project root
  it is `<slug>`; in a shared root it is a project-qualified basename such as
  `<project>-<slug>`. Docs-cli archives into `archive/<date>/` by flattened
  basename, so directories alone cannot prevent two projects' same-slug
  artifacts from colliding. Reject a new stem that is not root-globally unique
  across live and archived milestone artifact basenames.
- Reject a new slug that ends in `-impl` or `-test-matrix`, or whose derived
  `<project-path><artifact-stem>.md`,
  `<project-path><artifact-stem>-impl.md`, or
  `<project-path><artifact-stem>-test-matrix.md` path collides. Validate the
  three corresponding basenames root-globally as well as their live paths.
- Activation freezes identity. A `planned` row may be renamed through the
  managed path; after its first transition to `active`, never rename the slug
  or physical artifact stem. Later wording changes update the title. A genuine
  identity change cancels or supersedes the old milestone and creates a new
  row.
- Existing numeric or hierarchical slugs remain valid during migration. Never
  mass-rename activated, paused, completed, cancelled, or archived work. For a
  materialized legacy row, the tracker link target records its frozen physical
  stem even when it does not follow the new project-qualified convention.

Derived companion paths are `<project-path><artifact-stem>-impl.md` and
`<project-path><artifact-stem>-test-matrix.md`. The branch stack remains
`<project-branch-prefix><slug>/{milestone-setup,phases-1-4,phases-5-10,simplify}`.
Set `<project-branch-prefix>` to `<project>/` whenever the repository can
contain more than one suite project; use the empty string only when it
guarantees one project-wide milestone namespace. Freeze the resolved prefix at
first activation. Before creating a branch, reject a duplicate `Project:` slug
or any existing branch root owned by another project; never repair a namespace
collision by switching or renaming an activated milestone's prefix.

## State, eligibility, and claiming

Normal transitions are:

```text
planned -> active -> complete
             |
             v
           paused -> active

planned | active | paused -> cancelled
```

`paused` is a resumable post-activation hold, so its slug is already frozen.
Never silently treat an `active` row as stale; inspect its artifacts and branch
state, then resume or explicitly reconcile it.

A row is eligible only when all four are true:

1. its stored state is `planned`;
2. the project Definition of Ready passes;
3. every slug in `Depends on` has stored state `complete`; and
4. a materialized milestone document has no live `blocked-by` edge.

If an unmaterialized planned row needs a live blocker, do not invent a stored
`blocked` state or silently create a stub. Materialize an intentional planning
stub through the normal docs workflow first, or express a real milestone
prerequisite in `Depends on`.

`next` is the lexicographically first eligible slug at the lowest eligible
`Order`. An explicitly named eligible milestone may be selected out of normal
order. If no row is eligible, report the stored states, unmet dependencies,
live blockers, and DoR result; do not perform cycle analysis or act as a graph
scheduler.

Claim before production delegation or branch creation:

1. inspect and validate the tracker;
2. choose exactly one eligible `planned` row;
3. immediately re-read the tracker before writing;
4. if the table changed, write nothing and require reconciliation;
5. change only that row to `active`, preserving its `Order` and slug; and
6. `docs touch <project-path>milestone-plan.md`, then run `docs check .` before
   delegating.

Serialize selection and claim through one conductor/current checkout. This
contract does not pretend to provide atomic claims across independent
worktrees; a demonstrated need for that belongs in a later utility, not in
ad-hoc lock files or extra tracker columns.

## Ordering and relationships

Insert a cohort with any available intermediate integer. When an interval is
exhausted or the table becomes hard to scan, normalize distinct cohorts back to
`100`, `200`, `300`, and so on. Preserve equal-value cohorts and every slug.

For materialized milestone documents, synchronize these relationships:

- Every document in one distinct `Order` cohort has `precedes` edges to every
  materialized document in the next higher distinct `Order` cohort; the CLI
  writes the reciprocal `follows` halves. "Next higher" means the smallest
  stored `Order` value greater than the current value, even when no document in
  that cohort has been materialized. In that case, write no forward sequence
  edges and do not skip to a later cohort. Same-cohort peers have no sequence
  edge.
- Every materialized dependency in `Depends on` is represented by
  `depends-on`; the CLI writes the reciprocal `required-by` half.
- A condition preventing work now uses `blocked-by`; the CLI writes the
  reciprocal `blocks` half. Remove that pair when the condition clears without
  erasing durable dependency history.

Use `docs relate add/remove` with root-relative, `<project-path>`-qualified
source and target operands for all six recognized verbs. Never hand-author one
half. When either endpoint is archived, pass the required one-line `--reason`;
docs-cli will preserve archive history and append its revision audit. Sequence,
dependency, and blocker edges are navigation and scheduling facts, never
archive authorization.

After insert, reorder, normalization, materialization, planned rename, or
dependency change, derive the complete desired edge set, remove obsolete
recognized pairs, add missing pairs, and run `docs check .`. Do not touch
free-form relationships or infer dependency from sequence.

## Operation boundaries

- **Initialize:** create the tracker with `docs new plan
  <project-path>milestone-plan` and the exact table. It starts
  `Lifecycle: draft`; promote it to `active` only after the initial table,
  project scope, Definition of Ready, and dependencies validate. Update
  `<project-path>status.md` to preserve its narrative summary, add the relative
  body link `[Milestone Plan](milestone-plan.md)`, and remove any copied rows or
  independent schedule/next claims. If it is absent, create it with `docs new
  status <project-path>status` and promote it to `Lifecycle: active`. Then
  atomically run
  `docs touch <project-path>milestone-plan.md <project-path>status.md` before
  `docs index .` and `docs check .`. Foundation owns normal initialization.
- **Insert:** add one project-unique semantic row. Use a plain cell unless an
  intentional document already exists.
- **Materialize:** resolve and root-globally validate `<artifact-stem>`, then
  create `<project-path><artifact-stem>.md`,
  `<project-path><artifact-stem>-impl.md`, and
  `<project-path><artifact-stem>-test-matrix.md`. Replace the plain cell with a
  relative link whose text is `<slug>` and whose target is
  `<artifact-stem>.md`, then synchronize sequence and dependency relationships.
- **Reorder/normalize:** change only `Order` and derived sequence edges.
- **Pause/resume/cancel:** apply the allowed state transition; record the
  required reason. Resuming returns `paused` to `active`, not `planned`.
- **Rename:** allowed only while `planned`. Update dependency cells, use
  `docs mv` with `<project-path>`-qualified operands for every materialized
  milestone/companion path, recompute and revalidate the artifact stem, and
  resynchronize relationships. Reject after activation.
- **Dependency/blocker:** update `Depends on` for durable milestone
  prerequisites; use relationship pairs for materialized endpoints. Blocker
  changes never change the five stored states.
- **Complete:** after implementation, review, simplification, exact-scope
  archive, and final checks succeed, change `active` to `complete`. The archive
  operation uses primary `<project-path><artifact-stem>.md` and literal scope
  `'<project-path><artifact-stem>-*'`; its preview must select only the two
  canonical companions. Docs-cli flattens all three basenames into the dated
  archive, then rebases the tracker link; preserve that link.

Every body edit is followed by `docs touch`; every relationship operation uses
`docs relate`; every batch ends with `docs index .` and `docs check .`.
Initialization and migration explicitly own the status-body normalization
above and touch the qualified tracker and status paths together. Other tracker
operations do not edit status merely to cache a newly derived `next` result.

## Proposal-first migration

When an existing `<project-path>milestone-plan.md` lacks the canonical table,
inspect the plan, `<project-path>status.md`, milestone and implementation logs,
branches, lifecycle, archive paths, and relationships. Produce a complete
proposed mapping before writing:

- retain every existing slug unless a still-`planned` row is explicitly
  approved for semantic rename;
- retain each materialized activated or historical row's physical artifact
  stem from its link/path rather than renaming it to the new convention;
- map distinct existing plan positions to `100`, `200`, `300`, preserving
  evidenced parallel cohorts with equal values;
- infer state from all available evidence rather than one field;
- map durable prerequisites separately from sequence and live blockers; and
- identify live-path and root-global flattened-basename collisions across the
  milestone and both companions.

Explain every ambiguous order, state, dependency, identity, project-path, or
project mapping under the shared operator-interaction policy and obtain
confirmation. Apply only the confirmed plan. Normalize the status body to keep
its project narrative and the same-directory link
`[Milestone Plan](milestone-plan.md)` while removing copied tracker rows and
independent schedule/next claims. If status is absent, include creation of
`<project-path>status.md` with `docs new status <project-path>status` and
promotion to `Lifecycle: active` in the confirmed proposal. Then synchronize
relationships, atomically `docs touch` the qualified tracker and status paths,
and run `docs index .` followed by `docs check .`. Do not create milestone stubs,
mass-rename history, or add cycle detection during migration.
If frozen activated or historical legacy stems collide, record the exact
flattened archive collision as a delivery blocker for explicit operator
resolution; migration itself must not rename those paths.

## Validation

Fail closed when any of these is false:

- the canonical table has exactly the five named columns;
- every order is an integer and every state is allowed;
- each project has unique, collision-free slugs and companion paths;
- every new physical artifact stem and its companion basenames are
  root-globally unique, while activated legacy stems remain frozen;
- paused and cancelled rows have reasons;
- every dependency names a tracker row and no row depends on itself;
- every linked milestone path exists;
- materialized adjacent cohorts and dependencies match their recognized
  reciprocal relationship pairs;
- `<project-path>status.md` links to `milestone-plan.md` without duplicating its
  rows or independently declaring schedule or next work; and
- `docs check .` succeeds, including reciprocal metadata and local body links.

Cycle analysis and broader scheduling optimization are deliberately excluded
from version one.
