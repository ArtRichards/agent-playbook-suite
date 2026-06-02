---
name: project-foundation
description: Bootstrap a new project's foundation — charter, scope, architecture, milestones, definition-of-ready — as a docs-managed Markdown tree. Use when starting a project or sub-project that needs front-half planning before any implementation. Triggers on "start a project", "create a foundation", "scope a new project", "draft a charter", "plan a project from scratch". Authors every artifact via `docs new` so metadata, lifecycle, and the inter-doc graph are correct by construction.
---

# project-foundation

Drive the front-half foundation flow — charter through Definition
of Ready — across ten phases, producing a docs-managed Markdown
tree the `create-milestones` skill can pick up.

Requires [`docs-cli`](https://github.com/ArtRichards/docs-cli)
**v1.4.0 (M10+)** or later. The workflow relies on the
`Lifecycle:` metadata convention, `docs new --body-from -|<path>`,
tree-wide `[exclude]` rules in `.docs.toml`, and atomic
multi-file `docs touch <file>...`. Ask the user to upgrade if the
host's `docs` binary is older.

## When this applies

The user wants to start a new project (or a sub-project) and needs
the foundation work done before any implementation begins:
problem, scope, architecture choice, milestone plan, environment,
data, test strategy, documentation plan, risks, Definition of
Ready gate.

Do **not** apply when:

- The project already has a docs tree with foundation artifacts —
  use `create-milestones` to begin or resume milestone work.
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

**Read the playbook before authoring any artifact.**

## Invariants

These are non-negotiable across the whole flow:

1. **Every doc is *created* with `docs new`.** Never hand-author
   the metadata block. After creation, only two metadata fields
   are touched in place: `Lifecycle:` (when a doc advances
   draft → active or similar), and `Related:` (when adding typed
   edges). Every other field is owned by the CLI; use `docs touch`
   to bump `Updated:`.
2. **The controlled-vocab lifecycle field is `Lifecycle:`** (M7+).
   A controlled-vocab `Status:` line is wrong; `docs check` exits
   2. (A free-form `Status: <prose>` body line is allowed but this
   skill prefers `Lifecycle: blocked` + `Related: blocked-by:` for
   blocked states — queryable, not prose.)
3. **`docs new --body-from -` is the standard authoring shape.**
   One Bash call, no Read-before-Write friction. Never include a
   metadata block in piped body content.
4. **`docs check --stale 14` is the mechanical gate** at every
   phase boundary and at Definition of Ready.
   - Exit 0 → mechanically clean; apply qualitative review.
   - Exit 1 → warnings (medium-confidence inferences, stale docs);
     review and `docs touch` or update content; re-run.
   - Exit 2 → errors (missing fields, broken refs, lifecycle/
     location drift); fix before proceeding.
5. **Never hand-edit `INDEX.md`.** The `<!-- docs:generated -->`
   block is rewritten by `docs index`, `docs touch`, `docs
   archive`, and `docs mv`. (`docs new` does *not* regenerate it;
   call `docs index <root>` after a batch of `docs new` calls, or
   let the next `docs touch` do it.)
6. **One phase at a time.** Ask the phase's questions, wait for
   answers, author artifacts, confirm before moving on.

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
| foundation-log | `log` | active (ongoing) |
| definition-of-ready | `reference` | draft → active when green |
| status | `status` | active (lives the project's life) |

## Hand-off to create-milestones

When DoR flips to `active`:

1. `docs check <root> --stale 14` exit 0.
2. `docs list --lifecycle active --project <p>` shows every
   foundation artifact in the expected state.
3. Agent context handled in Phase 8 — `CLAUDE.md` and/or
   `AGENTS.md` scaffolded fresh from the template, companion
   additions written next to existing files for operator review,
   or confirmed already-complete.
4. Update `status.md`'s "Current milestone" to point at M1;
   `docs touch status.md`.
5. Suggest the user invoke `create-milestones` to begin M1 setup.

## Per-phase output format

1. State the phase name and objective.
2. Ask the phase's questions (from the playbook).
3. Wait for answers.
4. Author the artifact(s) with `docs new --body-from -`.
5. Append a one-line entry to `foundation-log.md`; `docs touch`.
6. Confirm before moving to the next phase.
