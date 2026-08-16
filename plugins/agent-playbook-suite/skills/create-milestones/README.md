# create-milestones

An agent workflow skill that selects, creates, advances, and completes
semantic milestones for a project whose foundation work is done.

Drives the risk-aware 10-phase TDD methodology (Define Contract → Write Tests
RED → Create Data/Fixtures → Run Tests RED Baseline → Update Base Interfaces →
Implement Offline/Core Path → Update Tool/Wrapper Layer → Run Tests GREEN →
Integrate / Accept / Dogfood → Quality/Docs/Refactor). The canonical
`milestone-plan.md` tracker owns semantic identity, order, execution state,
dependencies, and next-work selection. The skill claims one eligible row,
derives a physical `<artifact-stem>`, and authors its task plan,
implementation log, and test-matrix companion via
[`docs new`](https://github.com/ArtRichards/docs-cli), then previews and archives
the explicit set with
`docs archive <milestone-path> --cascade-only '<archive-scope>'` on completion.

## What it produces

- A milestone task-plan doc (`<project-path><artifact-stem>.md`, `Role:
  milestone`) capturing the risk level, behavior contract, test strategy,
  10-phase plan, decisions, deliverables, success criteria, and per-phase
  checklist.
- A milestone implementation log (`<project-path><artifact-stem>-impl.md`,
  `Role: log`, paired via `Related: pairs-with`) updated phase by phase.
- A test matrix (`<project-path><artifact-stem>-test-matrix.md`, `Role: spec`)
  mapping contract clauses to visible product
  tests or explicit non-product checks, hidden/generalization categories,
  adequacy checks, and mock-audit notes.
- A linked tracker row that moves `planned` → `active` before artifact creation
  and `active` → `complete` only after exact-scope archive and validation.
- A narrative `status.md` update reflecting current work and phase without
  duplicating tracker rows or independently storing next work.
- An archived milestone set on completion — task plan, log, and test matrix
  moved under `archive/YYYY-MM-DD/` only after the same
  `--cascade-only '<archive-scope>'` scope is previewed and confirmed to select
  exactly the intended companions. Relationships expose candidates; they
  never grant archive permission.

Before completion writes, the workflow classifies the live/archive/tracker
state. If docs-cli already moved exactly the three expected artifacts but the
tracker row is still `active`, it verifies archive dates, lifecycle metadata,
reason, links, and `docs check`, then resumes only the tracker and status
updates. If the verified row is already `complete`, it reports completion after
a final check. Mixed or unexplained state stops rather than replaying archive
or editing archived files.

## When to invoke

Triggers on "create a milestone", "start session-storage", "next milestone",
"begin implementation", and "advance the project". Existing activated numeric
or hierarchical slugs remain valid migration history, but new work uses stable
semantic slugs and tracker-owned order. Use after
[`project-foundation`](https://github.com/ArtRichards/project-foundation) has
set up the docs tree.

For end-to-end autonomous milestone execution (planning + implementation + review + simplify in one go), use [`ship-milestone`](https://github.com/ArtRichards/ship-milestone) instead — it spawns fresh sub-agents per step and conducts the whole milestone.

After the contract and RED baseline are stable, the suite invokes
[`explore`](https://github.com/ArtRichards/agent-playbook-suite/tree/main/plugins/agent-playbook-suite/skills/explore)
only when repository evidence leaves a genuinely uncertain implementation
route or a selected route has been concretely invalidated.

## Install

Install this skill through the Agent Playbook Suite marketplace plugin. The
repository root `README.md` carries the Codex and Claude Code marketplace
commands. This directory is the portable skill payload for hosts that support
direct skill-directory installs.

## Dependencies

- [`docs-cli`](https://github.com/ArtRichards/docs-cli) 2.0 or newer — prefer
  the newest release; required for `Lifecycle:`, `docs new --body-from`, multi-file
  `docs touch`, reciprocal `docs relate`, body-link rebasing/checking, and safe
  explicit archive selection. Install with `pip install --upgrade docs-cli`.
- Companion skills (recommended):
  [`project-foundation`](https://github.com/ArtRichards/project-foundation)
  (run first), the thin `manage-milestone-tracker` workflow for independent
  tracker mutations, conditional `explore`,
  [`ship-milestone`](https://github.com/ArtRichards/ship-milestone),
  [`sync-and-commit`](https://github.com/ArtRichards/sync-and-commit) (called at
  phase/step boundaries), and [`simplify`](https://github.com/ArtRichards/simplify)
  (Phase 10). Tracker management uses docs-cli primitives; the suite does not
  add a separate tracker utility.

## Convention

Follows the docs-cli convention: each Markdown file is self-describing via a
metadata block under the H1. The tracker is the authority for semantic identity,
order, state, and dependencies; task plan, impl log, and test matrix are linked
with `Related: pairs-with` so they appear as one-hop archive candidates. The
explicit previewed scope, not those relationships, authorizes the archive.
The tracker uses `<slug>` as semantic identity. For new work in a dedicated
docs root, `<project-path>` is empty and `<artifact-stem>` equals `<slug>`, so
the familiar primary and scope are `<slug>.md` and `'<slug>-*'`. In a shared
root, resolve `<project-path>` to the root-relative project directory plus
trailing slash (for example `specs/payments/`) and derive `<artifact-stem>` as
`<project>-<slug>`. Use both on every docs-cli path operand,
including the primary, companions, tracker, status,
relationship endpoints, and archive scope. This permits separate projects to
reuse a semantic slug and archive it on the same date without a basename
collision; neither the project path nor artifact stem replaces tracker
identity. Same-directory Markdown links may remain relative. Established
activated and historical filenames stay frozen. If such legacy paths produce
an actual occupied archive destination, the workflow stops instead of
falsifying the date or renaming a frozen identity.
When operator input is genuinely needed, the suite explains the evidence,
reason for asking, practical effects, an example, and a recommendation when one
is supportable. See
[`docs-cli`'s convention spec](https://github.com/ArtRichards/docs-cli/blob/main/docs/convention.md).

## License

MIT — see [`LICENSE`](LICENSE).
