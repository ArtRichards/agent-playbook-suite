# Milestone Playbook

Step-by-step procedure the `create-milestones` skill drives.
Short trigger lives in [`../SKILL.md`](../SKILL.md); substance
lives here.

The 10-phase TDD reference lives next door in
[`tdd-phases.md`](tdd-phases.md). The shared risk and adequacy
policy lives at
[`../../_shared/references/agentic-quality-model.md`](../../_shared/references/agentic-quality-model.md).
The canonical milestone tracker contract lives at
[`../../_shared/references/milestone-tracker.md`](../../_shared/references/milestone-tracker.md).
The shared operator-question policy lives at
[`../../_shared/references/operator-interaction.md`](../../_shared/references/operator-interaction.md).
Read all four before selecting, creating, driving, or completing a
milestone.

## When this applies

The user wants to create, advance, or complete a milestone for
a project whose foundation work is complete (the Definition of
Ready doc is `Lifecycle: active`).

## Project-qualified path model

Resolve the docs root and owning project directory once, before running any
docs-cli write. Run the displayed root-relative commands from the docs root;
`--root <root>` selects configuration but does not rebase every positional
file operand from another working directory:

- `<project-path>` is empty when the project's docs are flat in a dedicated
  root. In a shared root it is the root-relative directory that contains this
  project's tracker, with a trailing slash, such as `specs/payments/`.
- `<artifact-stem>` is the physical basename without `.md`. For a new
  milestone, set it to `<slug>` in a dedicated root and `<project>-<slug>` in
  a shared root. Before using it, reject a root-global collision with another
  milestone artifact stem or any of its three derived basenames.
- For any materialized or ever-activated milestone, recover
  `<artifact-stem>` from its established tracker link and artifact paths.
  Preserve that stem even when it is a legacy `<slug>` basename; never
  mass-rename activated or historical work. An unmaterialized cancelled row
  has no artifact stem to recover.
- `<milestone-path>` = `<project-path><artifact-stem>.md`.
- `<impl-path>` = `<project-path><artifact-stem>-impl.md`.
- `<matrix-path>` = `<project-path><artifact-stem>-test-matrix.md`.
- `<tracker-path>` = `<project-path>milestone-plan.md`.
- `<status-path>` = `<project-path>status.md`.
- `<archive-scope>` = `<project-path><artifact-stem>-*`, passed as one
  shell-quoted argument.

Use these root-relative paths for every `docs new`, `docs touch`, `docs
archive`, `docs mv`, and `docs relate` operand. Apply the same project path to
other project docs such as `definition-of-ready.md`, `followup-log.md`, and
`feedback-log.md`. `<slug>` remains semantic tracker identity and the branch
component; `<artifact-stem>` is physical identity that keeps dated archive
basenames root-global. Same-directory Markdown body links and tracker cells
may stay basename-relative, for example
`[<slug>](<artifact-stem>.md)`. `Related:` metadata targets are root-relative
and use the qualified paths.

In the dedicated-root worked example below, `<project-path>` is empty and
`<artifact-stem>` equals `fetch-and-parse`, so the shown commands remain
directly runnable.

## Bootstrap (Step 0) — verify the foundation

1. **Locate the docs root.** Walk up from the project location
   for `.docs.toml`. If absent → redirect to
   `project-foundation`. (Do not bootstrap a docs root here.)
2. **Verify the Definition of Ready is active:**
   ```sh
   docs list --root <root> --project <p> --role reference --lifecycle active
   ```
   Output must include `<project-path>definition-of-ready.md`. If it shows
   `draft` or is missing → redirect to `project-foundation`.
3. **Verify mechanical hygiene.** `docs check . --stale 14`
   must exit 0 or 1. Exit 2 is any hard error — lifecycle drift,
   broken refs, and as of docs 2.0 also `missing-inverse`,
   `broken-body-link`, `outside-root-body-link`, `duplicate-field`
   and `archive-date-drift`. Fix before adding milestone work. A
   tree upgraded from docs 1.x can surface pre-existing damage here
   on its first run; that is the point, and the docs-cli CHANGELOG's
   *Upgrading from 1.x* section carries a repair recipe per rule.
4. **Read the milestone-plan.**
   `docs list --root <root> --project <p> --role plan --lifecycle active`
   should show `<tracker-path>`. Validate its exact
   `Order | Milestone | State | Depends on | Notes` table against the
   shared tracker contract. If the project still has a legacy plan,
   run the proposal-first `manage-milestone-tracker` migration before
   selecting work; do not infer an id-based fallback here.
5. **Read `<status-path>`** for the latest narrative.
6. **Read `<project-path>use-cases.md`** when it exists — milestone tests focus
   primarily on the primary use cases recorded there, and the
   test matrix maps against them. If it is missing, suggest
   running the `use-cases` skill first (optional, but strongly
   preferred); do not block milestone work on it.

## Step 1 — Select and claim the milestone

`<tracker-path>`, not filename sorting or `<status-path>`, selects the
work. Use the tracker row's project-unique semantic slug unchanged.
For new rows this is a meaningful lowercase kebab-case identity such
as `session-storage`; existing activated numeric or hierarchical
slugs remain valid migration history and are never mass-renamed.

1. **Resolve the requested row.** For `next`, choose the
   lexicographically first eligible slug at the lowest eligible
   `Order`. A row is eligible only when it is `planned`, the project
   Definition of Ready passes, every slug in `Depends on` is
   `complete`, and a materialized milestone doc has no live
   `blocked-by` edge. An explicitly named eligible row may be selected
   out of normal order. If nothing is eligible, report the stored
   states, unmet dependencies, live blockers, and DoR result; do not
   create artifacts.
2. **Distinguish resume from creation.** An explicitly named `active`
   row is in-flight work: inspect its task plan, companions, phase
   record, branch state, and tracker relationships, then resume or
   reconcile it. If its link already points under the archive directory,
   route directly to Step 4's completion-state checkpoint; do not treat the
   missing live paths as permission to recreate them. Never treat age as
   evidence that the claim expired.
   An explicitly named `complete` row whose link points under the archive
   directory also routes to that checkpoint for read-only final verification;
   never recreate or re-archive it.
   A `paused` row must first be explicitly resumed to `active` through
   the tracker workflow.
3. **Validate identity before activation.** Reject reserved
   `-impl`/`-test-matrix` endings. For new work, scan the whole docs root,
   including archives, and reject any milestone artifact basename already
   using `<artifact-stem>.md`, `<artifact-stem>-impl.md`, or
   `<artifact-stem>-test-matrix.md`; also reject the three derived live-path
   collisions. Activation freezes both semantic slug and physical stem; later
   order changes never rename its files, branches, or links.
4. **Claim before production work.** Immediately re-read the canonical
   table. If it differs from the inspected snapshot, write nothing and
   require reconciliation. Otherwise change only the selected row's
   `State` from `planned` to `active`, preserving `Order`, slug,
   dependencies, and notes. Then run:

   ```sh
   docs touch <tracker-path>
   docs index .
   docs check . --stale 14
   ```

   Do not create the artifact set, delegate production work, or create
   a branch until this claim passes. This current-checkout sequence is
   deliberately serialized; it does not claim atomic coordination
   across independent worktrees.

Direct insert, reorder/normalize, planned rename, dependency, blocker,
pause/resume, or cancel requests belong to the thin
`manage-milestone-tracker` workflow backed by the shared contract.
It uses docs-cli primitives; do not add a separate tracker utility or
duplicate its algorithms here.

**Any plan change — insertion, re-scope, or cancellation — sweeps the
speculative ledger.** Check `<project-path>followup-log.md` for entries whose named
consuming milestone the change re-scopes or removes. Remove the now
dead reserved surface or re-justify it against a real consumer; never
leave silent rot.

**Sweep the project logs before authoring.** Read
`<project-path>followup-log.md` and `<project-path>feedback-log.md` for open
entries this milestone should incorporate. For each item taken
on, move its content into the milestone doc (contract, test
hooks, or deliverables as appropriate) and remove the entry from
the log — the logs hold open items only, and an incorporated item
needs no log mention.

In the same sweep, check speculative-ledger entries naming this
milestone as consumer (surface an earlier milestone built or
reserved "for" this one): if the milestone's work will consume the
surface, plan for it and close the entry at completion; if not,
challenge it — the reserved surface is a removal candidate, not a
default to build on.

The tracker row supplies the slug; do not invent or confirm a separate
ordinal id. Confirm the initial Risk Level (Lite / Standard / High)
with the user before authoring when project evidence does not already
settle it. Propose the level with plain-language
reasoning proportional to the risk per the shared quality model's
"Choosing a level" rule:
default Standard for ordinary work, Lite reserved for
low-blast-radius changes, and High only with an explicit trigger
from the model's High list — stated and explicitly approved by
the operator, never assigned unilaterally. Ground the confirmation
in the selected milestone-plan entry and the concrete risk signal;
explain what the level changes in the later gates.

## Step 2 — Create the milestone artifacts

Three project-qualified docs per milestone:

- task plan: `<milestone-path>`
- implementation log: `<impl-path>`
- test matrix: `<matrix-path>`

Treat the milestone's `Decisions` section and `<matrix-path>` as the
canonical homes for provenance. Nearby code comments or docs may also carry
references. Make every reference resolve to its source without guessing, using
the most readable path, link, or qualified identifier for the repository;
qualify the project and milestone when bare decision numbers could collide.
For tests written or actively modified during the milestone, keep display names
behavior-first and keep milestone, decision, phase, step, review, and amendment
references outside those names.

### 2a. Task plan — `Role: milestone`, `Lifecycle: draft`

```sh
docs new milestone <project-path><artifact-stem> --project <p> --title "<Title>" --body-from - <<'EOF'
## Overview

- Milestone: <slug>
- Title: <Title>
- Surface: <what this milestone delivers>
- Test Matrix: [<artifact-stem>-test-matrix.md](<artifact-stem>-test-matrix.md)

## Risk Level

Lite | Standard | High

Reason:

Selected gates:

## Contract

### Goal

<one paragraph — what success looks like>

### Scope

- In scope:

### Out of scope

- Not included in this milestone:

### Facts

- <known facts from the foundation, architecture, and milestone plan>

### Assumptions

- <assumption, owner, validation plan>

### Ambiguities

- <ambiguity, proposed default, needs operator decision? yes/no>

### Inputs

- API inputs:
- UI inputs:
- CLI inputs:
- Data/state preconditions:

### Outputs

- Return values / responses:
- UI states:
- Files/events/side effects:

### Invariants

- Must always:
- Must never:
- If X then Y:

### Error cases

- <error condition, expected behavior, observability>

### Non-functional constraints

- Performance budget:
- Security/privacy constraints:
- Compatibility/migration constraints:
- Reliability/observability needs:

### Acceptance examples

- Given:
- When:
- Then:

### Taste anchors

- Reference modules this change should read like:
- In-project libraries to reuse:
- Established patterns to follow (error handling, config, logging, naming):
- Expected diff footprint:
- Dependency policy: no new dependencies without a logged decision

### Forbidden shortcuts

- Do not special-case visible examples.
- Do not branch on test literals.
- Do not weaken valid tests or configured explicit checks merely to obtain GREEN. Correct erroneous expectations under the shared quality model's Check calibration rule; preserve the agreed behavior and meaningful coverage.

### Test hooks

- Visible tests to generate:
- Explicit non-product checks, outside default product-test discovery:
- Hidden/generalization categories to hold back:
- Property/stateful invariants:
- Mutation-sensitive logic:
- Fuzz targets:
- Benchmark/security/schema checks:

## Test Strategy For This Milestone

### Visible tests

<visible tests that will drive implementation; cite contract clauses where practical>

### Non-product checks

<planning, documentation, handoff, or workflow checks; keep outside default product-test discovery>

### Hidden/generalization categories

Do not include actual hidden/private cases here if the implementation agent can read this repository.

### Property/stateful invariants

### Mutation-sensitive logic

### Fuzz targets

### Benchmark/security/schema checks

### Mock policy

- New or expanded mocks:
- Justification:
- Real-path test covering the same behavior:

## Test Matrix

Link: [<artifact-stem>-test-matrix.md](<artifact-stem>-test-matrix.md)

## Deliverables

Each deliverable names its consumer — the end user, or a specific later
milestone. A deliverable that is only surface for later milestones needs
explicit justification (see the shared quality model's Demand-driven chains).

- [ ] Schemas/contracts — consumer:
- [ ] Implementation — consumer:
- [ ] Tests (RED → GREEN)
- [ ] Data/fixtures
- [ ] Documentation updates
- [ ] Test matrix and adequacy results

## Current State Analysis

- Existing code: <what exists>
- Missing: <gaps>
- Known issues: <bugs/debt>

## TDD Implementation Plan

Phases follow the canonical 10-phase TDD methodology. Fill in
per-phase objectives, files, and exit criteria below as the
plan solidifies — each phase section needs `Objective:`,
`Files:`, `Exit:`.

### Phase 1: Define Contract
- Objective:
- Files:
- Exit:

### Phase 2: Write Tests (RED)
- Objective:
- Files:
- Exit:

(...repeat for Phases 3-10 — full names in tdd-phases.md...)

## Phase Checklist

- [ ] Phase 1 — Define Contract
- [ ] Phase 2 — Write Tests (RED)
- [ ] Phase 3 — Create Data/Fixtures
- [ ] Phase 4 — Run Tests (RED Baseline)
- [ ] Phase 5 — Update Base Interfaces
- [ ] Phase 6 — Implement Offline/Core Path
- [ ] Phase 7 — Update Tool/Wrapper Layer
- [ ] Phase 8 — Run Tests (GREEN)
- [ ] Phase 9 — Integrate / Accept / Dogfood
- [ ] Phase 10 — Quality, Docs, Refactor

## Decisions

<key choices and rationale — append as you go>

## Success Criteria

<what must be true for completion>
EOF
```

### 2b. Implementation log — `Role: log`, `Lifecycle: active`

```sh
docs new log <project-path><artifact-stem>-impl --project <p> --title "<Title> — Implementation Log" --body-from - <<'EOF'
## Overview

Chronological log of work on `<slug>`. Append a section per
phase with objective, files changed, actions, test results,
decisions.

## Risk / Adequacy Tracking

- Risk level:
- Test matrix: [<artifact-stem>-test-matrix.md](<artifact-stem>-test-matrix.md)
- High-risk RED approval:
- Hidden/generalization handling:
- Mock audit:
- Adequacy results:
- Taste triage: (each review finding fixed or waived with reason; High-risk waivers operator-approved)

## TDD Phase Progress

| Phase | Progress |
|---|---|
| 1. Define Contract | Pending |
| 2. Write Tests (RED) | Pending |
| 3. Create Data/Fixtures | Pending |
| 4. Run Tests (RED Baseline) | Pending |
| 5. Update Base Interfaces | Pending |
| 6. Implement Offline/Core Path | Pending |
| 7. Update Tool/Wrapper Layer | Pending |
| 8. Run Tests (GREEN) | Pending |
| 9. Integrate / Accept / Dogfood | Pending |
| 10. Quality, Docs, Refactor | Pending |

## Phase 1 — Define Contract

_Not started._
EOF
```

### 2c. Test matrix — `Role: spec`, `Lifecycle: draft`

```sh
docs new spec <project-path><artifact-stem>-test-matrix --project <p> --title "<Title> — Test Matrix" --body-from - <<'EOF'
## Risk level

Lite | Standard | High

Reason:

## Check classification

Product tests belong in default test discovery. Non-product checks run only when selected as explicit workflow gates.

## Matrix

| Contract clause | Visible test | Hidden/generalization check | Property/stateful check | Mutation target | Fuzz/benchmark/security/schema note |
|---|---|---|---|---|---|
| Success path |  |  |  |  |  |
| Error path |  |  |  |  |  |
| Boundary |  |  |  |  |  |
| Idempotency/retry/state |  |  |  |  |  |
| Security/privacy/perf |  |  |  |  |  |

## Use-case mapping

Tests focus primarily on the project's primary use cases (from
`use-cases.md`, when the project has one). Map this milestone's
tests to the use cases they demonstrate; mirror the mapping in
the use-cases doc's test-matrix table.

| Use case | Covered by |
|---|---|
|  |  |

## Mock policy

- New mocks:
- Justification:
- Real-path test covering same behavior:

## Hidden-test handling

- Actual hidden cases are not written here if the implementation agent can read this repo.
- Categories:
- Owner:
- Commands or CI job names:

## Adequacy results

- visible_pass_rate:
- hidden_pass_rate:
- hidden_generalization_gap:
- changed-lines coverage:
- changed-files branch coverage:
- mutation score on touched modules:
- property/stateful result:
- fuzz result:
- benchmark delta:
- security/schema/migration result:
- skipped deep gates and follow-up: <reference open followup-log.md entries>
EOF
```

### 2d. Link the milestone artifacts and plan

Add `Related:` typed edges (careful Edit on the metadata block;
`Related:` is the only metadata field this skill extends after
`docs new`).

To `<milestone-path>`:
- `pairs-with: <impl-path>`
- `pairs-with: <matrix-path>`
- `child-of: <tracker-path>`
- `implements: <project-path>charter.md`
- any relevant `pairs-with:` (architecture, test-strategy, etc.)

To `<impl-path>`:
- `pairs-with: <milestone-path>`
- `pairs-with: <matrix-path>`

To `<matrix-path>`:
- `pairs-with: <milestone-path>`
- `pairs-with: <impl-path>`

Archive candidate discovery follows `pairs-with` and `child-of` one
hop — never transitively, and never the reciprocal verbs
(`precedes`/`follows`, `depends-on`/`required-by`,
`blocks`/`blocked-by`). A candidate is context for the preview, not
authorization to move it. Ensure the milestone doc has `pairs-with`
links to both companions so the explicit completion scope can select
them.

**The `child-of` refusal is directional.** At docs 2.0 the archive
verb refuses, at exit 2 with zero bytes written, when a still-active
document **outside the plan** declares `child-of` a document **the
plan would archive** — the "parent archived out from under a live
child" case. Only that direction refuses. The milestone doc's own
`child-of: <tracker-path>` edge is unaffected and should stay: the
child is the thing being archived and the parent lives on, which is
the normal shape.

What to avoid is the reverse: giving a document that **deliberately
outlives** the milestone — a long-lived release or publish log kept at
the tree root — a `child-of` edge to the milestone doc. Make it a
**pair** instead. Note that `pairs-with` alone does not keep it out of
the write: it makes the document a *candidate*, and the glob decides.
Choose a slug the milestone glob does not match, or narrow the glob.

### 2e. Materialize the tracker row and synchronize relationships

After `<milestone-path>` exists, replace that row's plain semantic
`Milestone` cell (or verify its intentional stub link) with
`[<slug>](<artifact-stem>.md)`. Preserve the claimed `active` state, `Order`,
`Depends on`, and `Notes` exactly.

Derive the complete desired recognized edge set from the tracker
contract, then use `docs relate add/remove` rather than hand-editing
either half:

- every materialized milestone in the next lower distinct `Order`
  cohort `precedes` this row, and this row `precedes` every
  materialized milestone in the next higher distinct `Order` cohort.
  A cohort still counts when none of its milestone documents exists:
  add edges only to materialized documents in that exact cohort and
  never skip across it to a more distant cohort;
- every materialized slug in this row's `Depends on` has a
  `depends-on` / `required-by` pair with this milestone; and
- existing live `blocks` / `blocked-by` pairs stay independent of
  durable dependencies and sequence.

For example:

```sh
docs relate add <earlier-path> precedes <milestone-path>
docs relate add <milestone-path> depends-on <dependency-path>
```

Resolve `<earlier-path>` and `<dependency-path>` to their exact current
root-relative paths from the tracker links or docs inventory. A live endpoint
in this project normally starts with `<project-path>`; an archived endpoint
uses its dated archive path. Never substitute an ambiguous bare basename in a
shared root.

Use `docs relate remove` for obsolete recognized pairs. When either
endpoint is archived, supply a concise `--reason`, such as
`--reason "Synchronize <slug> with milestone tracker"`; docs-cli owns
the reciprocal edit and archived revision audit. Do not infer a
dependency from order, and do not treat any relationship as archive
membership.

### 2f. Flip the milestone artifacts to active

Edit `Lifecycle: draft` → `Lifecycle: active` in `<milestone-path>`
and `<matrix-path>` if `docs new spec` created the matrix as a draft.
Update `<status-path>` as a narrative summary linking
to `<tracker-path>` and the current milestone/phase; do not copy the
tracker rows or independently declare next work. Then:

```sh
docs touch <tracker-path> <milestone-path> <impl-path> \
  <matrix-path> <status-path>
docs index .
docs check . --stale 14
```

## Step 3 — Walk the TDD phases

Drive phases 1-10 one at a time. Full per-phase procedure in
[`tdd-phases.md`](tdd-phases.md). The cadence per phase:

1. State phase number, name, objective.
2. Inspect the plan, implementation log, test matrix, and relevant
   project surface. If the phase still needs clarification, ask only
   the unresolved question and follow the shared operator-interaction
   policy.
3. Do the work (code, tests).
4. Append a phase section to the impl log via body edit:

   ```markdown
   ## Phase N — <Phase Name> (<YYYY-MM-DD>)

   ### Objective
   <what this phase accomplished>

   ### Files Changed
   | File | Action | Notes |
   |------|--------|-------|

   ### Actions Taken
   - <bulleted list>

   ### Test Results
   - <test counts, pass/fail, notes>

   ### Risk / Adequacy Notes
   - Risk level:
   - Gate evidence:
   - Hidden/generalization handling:
   - Mock changes:
   - Adequacy gaps or follow-ups: <open items go to followup-log.md; reference them here>

   ### Issues/Decisions
   - <problems, decisions, rationale>
   ```

5. Flip that phase's progress-table cell from `Pending` to
   `Complete`.
6. Tick the matching `[ ]` → `[x]` in the milestone doc's
   Phase Checklist.
7. Update `<matrix-path>` when contract clauses,
   visible tests, hidden/generalization categories, adequacy
   checks, or mock policy change.
8. Update `<status-path>`'s narrative with the current milestone and
   phase. Keep it linked to `<tracker-path>`; do not duplicate
   tracker rows or independently declare next work.
9. `docs touch <milestone-path> <impl-path> <matrix-path>
   <status-path>`.
10. `docs index .` → `docs check . --stale 14`.
   Exit 2 → fix before next phase.
11. Confirm with the user before starting the next phase. Summarize
    the completed phase evidence, explain what the next phase will
    change, and recommend proceeding when the exit criteria are met.

## Step 4 — Complete the milestone

When Phase 10 wraps up, run this **completion-state checkpoint before any
write**. Read `<tracker-path>`, list this project's live and archived docs, and
classify the three derived artifact paths:

- **Normal live completion:** exactly `<milestone-path>`, `<impl-path>`, and
  `<matrix-path>` are live, the row is `active`, and its link resolves to
  `<milestone-path>`. Continue with item 1 below.
- **Archive already applied, tracker still active:** no live copy exists; the
  same three expected basenames, with the owning `Project: <p>`, are together
  in one dated archive directory; and the row is `active` with a link rebased
  to that archived milestone. Verify all archive witnesses before resuming:
  each file is `Lifecycle: archived`, each `Archived:` value equals its parent
  directory date, the primary has the expected non-empty `Archived-reason:`,
  the milestone completion summaries and final matrix evidence are present,
  metadata/body links resolve, no additional artifact from this project and
  scope was archived with the set, and `docs check . --stale 14` exits 0
  or 1. Then skip items 1-6 and resume at item 7. Do not run preview/apply
  again, recreate a live copy, or edit any archived file.
- **Archive and tracker completion already applied:** require the same exact
  archived set and witnesses, with the row already `complete`. Re-read the
  tracker, verify `<status-path>` has the completion/current-activity narrative
  and relative tracker link without a stored next-work choice, and run the
  final `docs check` without touching archived files. If it exits 0, or exits
  1 with every warning reviewed, report the milestone complete; do not replay
  items 1-9.
- **Anything else:** stop fail-closed. A live/archived mixture, a missing or
  extra matching companion, different archive dates, wrong project metadata,
  a tracker link that does not resolve to the witnessed primary, or a state
  other than the two recovery cases is unexplained. Report the evidence and
  require reconciliation rather than guessing or mutating either side.

For the normal path:

1. **Append a completion summary** to the impl log:
   verification results (commands + test counts), files
   added/modified/removed, documentation updated, risk level,
   test-matrix status, adequacy results, hidden-generalization
   gap if available, mock audit, lessons learned, pre-existing
   issues found.
2. **Append a completion summary** to the milestone doc under
   `## Milestone-completion summary`.
3. **Update `<matrix-path>`** with final visible,
   selected hidden/generalization, property/stateful, mutation,
   fuzz/benchmark/security/schema, mock-audit, skipped-gate,
   and follow-up status.
4. **Sweep open items to the project logs.** The milestone docs
   are about to leave the active tree, and the logs are the
   single home for open items: move any still-open follow-up,
   adequacy gap, or skipped-gate item into
   `<project-path>followup-log.md` as a
   dated entry, and any unaddressed feedback or idea into
   `<project-path>feedback-log.md`. Drop milestone-doc mentions of log entries
   this milestone incorporated. Use the qualified paths when running `docs
   touch` on the logs.
5. **Preview the exact plan, then archive it:**
   ```sh
   docs archive <milestone-path> --cascade-dry-run --cascade-only '<archive-scope>'
   docs archive <milestone-path> --cascade-only '<archive-scope>' --reason "Milestone <slug> complete"
   ```
   Pass the **same glob** to the dry run. `--cascade-dry-run` on its
   own lists the candidate neighbourhood with nothing selected, so it
   rehearses a different operation than the one you are about to run.

   Candidate discovery walks `Related: pairs-with` and `Related:
   child-of` one hop; relationships provide context for the preview,
   never authorization to move a document. The explicit glob decides
   which candidates join the named primary. `<archive-scope>` matches the
   candidates' canonical root-relative paths, reaching `<impl-path>` and
   `<matrix-path>` while anchoring a shared-root selection to this project
   directory and physical artifact stem.
   It is **not** a guarantee that those two are the only matches:
   `<project-path><artifact-stem>-release-log.md` would match as well. When
   `<project-path>` is empty, the glob has no slash and matches that basename
   at any depth. The optional
   long-lived quality companion deliberately uses
   `<project-path>quality/quality-log-<artifact-stem>.md`; because its basename starts with
   `quality-log-`, this milestone scope cannot select it. A legacy
   `<project-path>quality/<slug>-quality-log.md` can match the bare
   dedicated-root scope; rename it before closeout with
   `docs mv <project-path>quality/<slug>-quality-log.md <project-path>quality/quality-log-<artifact-stem>.md`.
   Require the previewed
   plan to include the primary and select exactly the intended
   companions. If it selects anything else, stop and narrow the scope
   or repair the candidate relationships before applying the same
   scope.

   Also require every planned destination to be unoccupied. The
   `<project>-<slug>` artifact stem makes new shared-root milestones globally
   distinct even when projects use the same semantic slug. If an established
   legacy stem or another unexplained path still produces a destination
   collision, treat the atomic preflight refusal as a reconciliation blocker.
   Do not bypass it by falsifying the archive date or renaming an activated
   identity.

   **Bare `--cascade` is retired at docs 2.0**: it refuses at
   exit 2, writes nothing, and prints the replacement recipe.
   `--interactive` is retired the same way, so there is no
   prompt to answer any more — read the preview instead.

   Read the preview before writing. It also reports every
   still-active document that will be left pointing at the newly
   archived set (`strands`), which is information, not an error.
   Among **strand** outcomes the write refuses only for the "live
   child" case above; other preflight failures (an unreadable or
   malformed member, a bad `--date`, an already-archived primary)
   refuse on their own terms.

   Result: the explicitly selected milestone set moves to
   `<root>/<archive-dir>/<today>/`,
   with `Lifecycle: archived`, an `Archived:` date recorded on
   every moved document, referring links rebased, and INDEX
   regenerated.
6. **Validate the archive before completing the tracker row:**
   ```sh
   docs check . --stale 14
   ```
   Exit 0 or 1. Verify that docs-cli rebased the selected row's
   milestone link to the dated archive path and that the row is still
   `active`. Errors block the state transition.
7. **Complete the tracker row and refresh the narrative.** Change only
   that row's `State` from `active` to `complete`; preserve `Order`,
   semantic slug, dependencies, notes, and the rebased archive link.
   Derive the next eligible row from the tracker contract for the operator
   response. Update `<status-path>` with a concise completion/current-work
   narrative that links to `<tracker-path>`; do not add a duplicate
   milestone table or store the derived next value.
8. **Run the final docs gate:**
   ```sh
   docs touch <tracker-path> <status-path>
   docs index .
   docs check . --stale 14
   ```
   Re-read the tracker and recognized relationships. Exit 0 or 1 is
   required; fix any error before reporting the milestone complete.
9. **Ask the user** about the next milestone. Ground the question in
   the updated status and milestone plan, explain what starting it
   will open, and recommend the derived next eligible semantic slug
   when one exists.

## Quality gate commands (per phase, adapt to project)

Project-side gate — substitute your stack's configured commands:

```sh
make format    # or: black . / prettier --write .
make lint      # or: ruff check . / eslint .
make typecheck # or: mypy . / tsc --noEmit
make test      # or: pytest / npm test
```

Plus the docs-side gate when the project uses docs-cli:

```sh
docs check . --stale 14
docs index .
```

`docs check` proves the docs tree is mechanically valid. It does
not prove behavior, visible-test adequacy, hidden/generalization
coverage, or risk gates. Use the milestone's Risk Level and the
shared quality model to decide which Phase 8 and Phase 10 gates
should run, be marked not configured, or be approved as skipped.

## Wrapping up a phase or step — sync-and-commit

When a phase, step, or milestone is ready to commit, invoke the
`sync-and-commit` skill. It runs the verification checklist
(consistency / completeness / accuracy), syncs the docs tree
(touches, indexes, checks), and commits with the project's
conventions from project context. Push happens only on feature/
milestone branches with a remote — never on `main`.

`ship-milestone` invokes sync-and-commit automatically at the
end of each step; for interactive use here, invoke it manually
when the phase or milestone is ready.

## Bug fixes and features inside a milestone

- **Feature inside a milestone:** add a section to the
  milestone's task plan; track in the impl log under the
  relevant phase. No new doc.
- **Bug fix:** scoped → impl log under that milestone.
  Cross-cutting → `docs new postmortem <project-path><slug> --project <p>`
  (incident retrospective) or
  `docs new decision <project-path><slug> --project <p>` (codified fix choice).
- **Hot-fix outside the TDD flow:** log to
  `<project-path>decision-log.md` so the audit trail stays complete.
- **Operator feedback, ideas, scope thoughts:** append a dated
  entry to `<project-path>feedback-log.md` (following the
  template embedded there) rather than burying them in phase
  notes or a milestone's follow-on sections. Engineering
  deferrals go to `<project-path>followup-log.md` the same way.

## Parallel milestones

Supported (rare):

- Equal `Order` values in the tracker define a parallel cohort. Each
  `next` invocation still deterministically selects and claims one
  eligible row; equal-order peers remain eligible for later claims.
- Each milestone is independent: separate `<milestone-path>`,
  `<impl-path>`, and `<matrix-path>` docs, all
  `Lifecycle: active`.
- `<status-path>` may narrate the active set but must not duplicate the
  tracker table or store its own next-work decision.
- `docs list --role milestone --project <p> --lifecycle active`
  shows the parallel set.
- Archive each with its own previewed `'<archive-scope>'` scope, validate the
  result, then change only that tracker row to `complete`.

## Worked example: link-checker `fetch-and-parse`

Continues the worked example from `project-foundation`: a small
fictional CLI that crawls a website and reports broken links. The
foundation and DoR are active. The tracker starts with semantic rows:

```markdown
| Order | Milestone | State | Depends on | Notes |
|---:|---|---|---|---|
| 100 | fetch-and-parse | planned | — | HTTP fetch, link extraction, in-memory crawl |
| 200 | persistence | planned | fetch-and-parse | Save crawl results |
```

Working directory: `~/code/link-checker/docs/specs/`.

### Bootstrap, select, and claim

```sh
docs list --root . --project link-checker --role reference --lifecycle active
# definition-of-ready.md ... reference ... active

docs list --root . --project link-checker --role plan --lifecycle active
# milestone-plan.md ... plan ... active

docs check . --stale 14
# exit 0
```

`fetch-and-parse` is the lexicographically first row in the lowest
eligible order cohort. It is `planned`, has no dependencies or live
blocker, and the DoR passes. After re-reading the unchanged table, the
workflow changes only its state to `active`:

```markdown
| 100 | fetch-and-parse | active | — | HTTP fetch, link extraction, in-memory crawl |
```

```sh
docs touch milestone-plan.md
docs index .
docs check . --stale 14
```

Standard risk is recommended because this is a customer-facing
network/parsing path but does not touch auth, billing, migration, or
data integrity.

### Create and materialize the artifacts

Step 2's templates create:

- `fetch-and-parse.md`;
- `fetch-and-parse-impl.md`; and
- `fetch-and-parse-test-matrix.md`.

The contract covers `link_checker.fetch`, `link_checker.parse`, and an
in-memory `link_checker.crawl` orchestrator. Given a seed URL, the CLI
BFS-crawls same-origin links to depth N and returns
`(url, status_code, depth)` tuples. Persistence is explicitly owned by
the later `persistence` row.

Add the companion `pairs-with` links and the milestone's
`child-of: milestone-plan.md` link. Replace the tracker cell with:

```markdown
| 100 | [fetch-and-parse](fetch-and-parse.md) | active | — | HTTP fetch, link extraction, in-memory crawl |
```

`persistence` is not materialized, so no sequence or dependency
relationship can be written to it yet. The tracker still records both
facts. Flip the task plan and matrix to `Lifecycle: active`, update the
status narrative, and validate:

```sh
docs touch milestone-plan.md fetch-and-parse.md fetch-and-parse-impl.md \
  fetch-and-parse-test-matrix.md status.md
docs index .
docs check . --stale 14
```

### Walk Phase 1

Phase 1 defines `Link`, `CrawlResult`, and `CrawlConfig` contracts and
records the Standard-risk test strategy. The implementation log entry
uses the normal phase structure:

```markdown
## Phase 1 — Define Contract (2026-05-25)

### Objective
Establish the behavior and data contracts for fetching and parsing.

### Files Changed
| File | Action | Notes |
|------|--------|-------|
| src/link_checker/models.py | added | Link, CrawlResult, CrawlConfig contracts |

### Test Results
N/A — Phase 2 owns tests.

### Risk / Adequacy Notes
- Risk level: Standard.
- Gate evidence: contract and initial test matrix drafted.
- Hidden/generalization handling: categories only, no private cases.
- Mock changes: none.
```

Flip the Phase 1 progress cell to `Complete`, tick its checklist item,
and update the status narrative to the current milestone and phase.
Then run:

```sh
docs touch fetch-and-parse.md fetch-and-parse-impl.md \
  fetch-and-parse-test-matrix.md status.md
docs index .
docs check . --stale 14
```

### Phases 2-10 (abridged)

The same cadence drives each phase. Phase 4 captures the intended RED
baseline; Phase 8 records GREEN and the Standard-risk gates; Phase 9
dogfoods the 20-page fixture site and records a 3.2-second crawl with
100% broken-link detection; Phase 10 finalizes the completion
summaries, test matrix, taste triage, and boundary sweep.

### Explicit-scope completion

The tracker row is still `active`. Preview the exact same scope that
will be applied:

```sh
docs archive fetch-and-parse.md \
  --cascade-dry-run \
  --cascade-only 'fetch-and-parse-*'
```

The plan includes the named primary and selects exactly the two
companion candidates. `milestone-plan.md` is reported but not selected;
relationships exposed it as context, not archive permission. Apply the
same scope:

```sh
docs archive fetch-and-parse.md \
  --cascade-only 'fetch-and-parse-*' \
  --reason "Milestone fetch-and-parse complete; fixture site crawled in 3.2s, 100% detection"
```

All three artifacts move to `archive/2026-06-12/`, and docs-cli
rebases metadata and Markdown body links, including the tracker cell.
Before changing tracker state:

```sh
docs check . --stale 14
```

After that check passes, change only the row state to `complete`:

```markdown
| 100 | [fetch-and-parse](archive/2026-06-12/fetch-and-parse.md) | complete | — | HTTP fetch, link extraction, in-memory crawl |
| 200 | persistence | planned | fetch-and-parse | Save crawl results |
```

`persistence` is now the derived next eligible row for the operator response.
Update `status.md` only with the completion/current-activity narrative and a
link to the tracker, not the derived next choice or a copy of the rows, then
validate the final state:

```sh
docs touch milestone-plan.md status.md
docs index .
docs check . --stale 14
```

### Post-archive

```sh
$ docs list --root . --project link-checker --role milestone
archive/2026-06-12/fetch-and-parse.md   milestone   archived

$ ls archive/2026-06-12/
fetch-and-parse-impl.md
fetch-and-parse-test-matrix.md
fetch-and-parse.md
```

The semantic identity stayed stable while its order and archive path
lived in the tracker. The artifact set is in the dated archive,
`persistence` remains an unmaterialized planned row, and `docs check`
is green.
