---
name: project-foundation
description: Bootstrap a new project's foundation — charter, scope, architecture, milestones, definition-of-ready — as a docs-managed Markdown tree. Use when starting a project or sub-project that needs front-half planning before any implementation. Triggers on "start a project", "create a foundation", "scope a new project", "draft a charter", "plan a project from scratch". Authors every artifact via `docs new` so metadata, lifecycle, and the inter-doc graph are correct by construction.
---

# project-foundation

Drive the front-half foundation flow — charter through Definition
of Ready — across ten phases, producing a docs-managed Markdown
tree the `create-milestones` skill can pick up.

Requires [`docs-cli`](https://github.com/ArtRichards/docs-cli) 2.0 or newer —
prefer the newest release. The workflow relies on the
`Lifecycle:` metadata convention, `docs new --body-from -|<path>`,
tree-wide `[exclude]` rules in `.docs.toml`, and atomic
multi-file `docs touch <project-path><file>.md...`. Recognized reciprocal milestone
relationships use `docs relate`, never hand-authored halves.

## When this applies

The user wants to start a new project (or a sub-project) and needs
the foundation work done before any implementation begins:
problem, scope, architecture choice, milestone plan, environment,
data, test strategy, documentation plan, risks, Definition of
Ready gate.

Do **not** apply when:

- The project already has a docs tree with foundation artifacts —
  use `manage-milestone-tracker` for tracker migration or direct tracker
  changes, and `create-milestones` to begin or resume milestone work.
- The user wants a one-off doc (a single charter, a single ADR) —
  author it with `docs new` directly.
- The user has an existing Markdown directory they want to adopt
  rather than start from scratch — point them at `docs migrate`
  first; this wizard runs after the tree is in convention.

## Substance lives in references/

This SKILL.md is intentionally short. The full procedure lives in:

- [`references/foundation-playbook.md`](references/foundation-playbook.md)
  — step-by-step procedure with a worked example.
- [`references/role-mapping.md`](references/role-mapping.md) — the
  artifact-to-role-and-lifecycle table.
- [`references/docs-toml-template.toml`](references/docs-toml-template.toml)
  — opinionated `.docs.toml` to drop at the bootstrapped docs root.
- [`references/claude-md-template.md`](references/claude-md-template.md)
  — the project-root `CLAUDE.md` template the wizard scaffolds
  (or proposes additions to) at Phase 8, with matching guidance
  for Codex `AGENTS.md`.
- [`../_shared/references/agentic-quality-model.md`](../_shared/references/agentic-quality-model.md)
  — shared risk levels, hidden/generalization policy, adequacy
  checks, and mock policy used by the full suite.
- [`../_shared/references/operator-interaction.md`](../_shared/references/operator-interaction.md)
  — shared requirements for grounded, understandable operator
  questions, clarifications, approvals, and decisions.
- [`../_shared/references/milestone-tracker.md`](../_shared/references/milestone-tracker.md)
  — canonical milestone identity, order, state, dependency, and
  relationship contract. Read it before Phase 4 or any tracker hand-off.

**Read the playbook and operator-interaction policy before authoring
artifacts or asking the operator questions. Read the milestone-tracker
contract before creating `milestone-plan.md`.**

The milestone-tracker contract is required. Before beginning the foundation
workflow, verify that the shared reference exists and read it in full. If it is
absent, stop and explain that the full Agent Playbook Suite installation is
incomplete; recommend reinstalling or updating the complete suite. Never
reconstruct the contract from the foundation playbook, project artifacts, or
another skill.

## Invariants

These are non-negotiable across the whole flow:

1. **Every doc is *created* with `docs new`.** Never hand-author
   the metadata block. After creation, only two metadata fields
   are touched in place: `Lifecycle:` (when a doc advances
   draft → active or similar), and free-form `Related:` entries.
   For the six recognized reciprocal verbs, use `docs relate add/remove`
   so both endpoints change together. Every other field is owned by the
   CLI; use `docs touch` to bump `Updated:`.
2. **The controlled-vocab lifecycle field is `Lifecycle:`.**
   A controlled-vocab `Status:` line is wrong; `docs check` exits
   2. (A free-form `Status: <prose>` body line is allowed.) Document
   lifecycle is distinct from milestone execution state: the canonical
   tracker stores only `planned`, `active`, `paused`, `complete`, or
   `cancelled`; a live `blocked-by` relationship makes blocking derived.
3. **`docs new --body-from -` is the standard authoring shape.**
   One Bash call, no Read-before-Write friction. Never include a
   metadata block in piped body content.
4. **`docs check . --stale 14` is the mechanical gate** at every
   phase boundary and at Definition of Ready.
   - Exit 0 → mechanically clean; apply qualitative review.
   - Exit 1 → warnings (medium-confidence inferences, stale docs);
     review and `docs touch` or update content; re-run.
   - Exit 2 → errors (missing fields, broken refs, lifecycle/
     location drift); fix before proceeding.
5. **Never hand-edit `INDEX.md`.** The `<!-- docs:generated -->`
   block is rewritten by `docs index`, `docs touch`, `docs
   archive`, and `docs mv`. (`docs new` does *not* regenerate it;
   call `docs index .` after a batch of `docs new` calls, or
   let the next `docs touch` do it.)
6. **One phase at a time.** Ask the phase's questions, wait for
   answers, author artifacts, confirm before moving on.
7. **Investigate before asking; propose, don't assign.** Inspect
   the repo or product before each phase and propose
   evidence-backed answers for the operator to correct. Risk
   levels in particular are proposed with reasoning and confirmed
   with the operator — default Standard; High only with a stated
   reason and explicit operator approval. If the operator asks,
   launch a thorough investigation of the entire product's
   foundation.
8. **Resolve one root-relative project path.** Use `<project-path>` as the
   directory prefix for every project artifact: empty for a dedicated project
   root, or a trailing-slash path such as `specs/<project-slug>/` when joining
   an existing shared root. Change the working directory to the docs root
   before every docs-cli operation; `--root` selects the tree but does not
   rebase relative document operands. Qualify every document operand and every
   `Related:` target with that prefix. Keep Markdown
   body links between same-directory project docs relative. A `Project:` is
   metadata ownership; `<project-path>` is live-tree disambiguation. For each
   milestone also validate a root-globally unique physical `<artifact-stem>`:
   `<slug>` in a dedicated root, `<project>-<slug>` in a shared root. Docs-cli
   flattens archive destinations to basenames, so this second layer prevents
   cross-project archive collisions while tracker identity remains `<slug>`.
   Require the kebab-case `Project:` slug to be repository-unique across every
   docs-managed tree; it is also the branch namespace when multiple suite
   projects exist.
   See Bootstrap (Step 0) and Phase 4 in the foundation playbook.

## Role mapping (summary)

Full table in [`references/role-mapping.md`](references/role-mapping.md).

| Artifact | Role | Lifecycle |
|---|---|---|
| charter | `charter` | draft → active |
| scope-and-constraints | `spec` | draft → active |
| stakeholders | `reference` | draft → active |
| options-comparison | `decision` | draft → active |
| architecture | `sketch` | draft → active (→ `reference` once authoritative) |
| decision-log | `log` | active (ongoing) |
| milestone-plan | `plan` | draft → active |
| env-and-tooling | `runbook` | draft → active |
| data-plan | `plan` | draft → active |
| test-strategy | `outline` | draft → active (→ `spec` later) |
| documentation-plan | `plan` | draft → active |
| risks | `log` | active (ongoing) |
| followup-log | `log` | active (ongoing) |
| feedback-log | `log` | active (ongoing) |
| foundation-log | `log` | active (ongoing) |
| definition-of-ready | `reference` | draft → active when green |
| status | `status` | active (lives the project's life) |

## Hand-off to use-cases, then create-milestones

When DoR flips to `active`:

1. Update `<project-path>status.md` with a narrative foundation-complete
   summary and the same-directory body link
   `[Milestone Plan](milestone-plan.md)`; remove copied tracker rows and
   independent schedule/next claims.
2. After all lifecycle and body edits, atomically touch every changed
   qualified path. The batch must include
   `<project-path>milestone-plan.md` and `<project-path>status.md`.
3. Run `docs index .` and require `docs check . --stale 14` to exit 0.
4. `docs list --root . --lifecycle active --project <p>` shows every
   foundation artifact in the expected state.
5. Agent context handled in Phase 8 — `CLAUDE.md` and/or
   `AGENTS.md` scaffolded fresh from the template, companion
   additions written next to existing files for operator review,
   or confirmed already-complete.
6. Run the `use-cases` skill with the resolved docs root, `<project-path>`, and
   `Project:` value — automatic after foundation completes; optional, but
   strongly preferred. It explores the primary use cases and records them in a
   use-cases doc that milestone test matrices map against.
7. Then suggest the user invoke `create-milestones` for `next milestone`
   or an explicitly named eligible semantic slug.

## Per-phase output format

1. State the phase name and objective.
2. Investigate what already exists (code, tests, configs), then
   ask the phase's questions (from the playbook), proposing
   evidence-backed answers where the investigation supports them.
3. Wait for answers.
4. Author the artifact(s) with `docs new --body-from -`.
5. Append a one-line entry to `<project-path>foundation-log.md`; run
   `docs touch <project-path>foundation-log.md`.
6. Confirm before moving to the next phase.
