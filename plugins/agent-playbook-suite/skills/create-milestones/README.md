# create-milestones

An agent workflow skill that creates, advances, and completes milestones for a
project whose foundation work is done.

Drives the risk-aware 10-phase TDD methodology (Define Contract → Write Tests
RED → Create Data/Fixtures → Run Tests RED Baseline → Update Base Interfaces →
Implement Offline/Core Path → Update Tool/Wrapper Layer → Run Tests GREEN →
Integrate / Accept / Dogfood → Quality/Docs/Refactor). Authors a `<slug>.md` task plan, `<slug>-impl.md`
implementation log, and `<slug>-test-matrix.md` companion per milestone via
[`docs new`](https://github.com/ArtRichards/docs-cli), and atomically archives
the set with `docs archive --cascade` on completion.

## What it produces

- A milestone task-plan doc (`Role: milestone`) capturing the risk level, behavior contract, test strategy, 10-phase plan, decisions, deliverables, success criteria, and per-phase checklist.
- A milestone implementation log (`Role: log`, paired via `Related: pairs-with`) updated phase by phase as work progresses.
- A test matrix (`Role: spec`) mapping contract clauses to visible product
  tests or explicit non-product checks, hidden/generalization categories,
  adequacy checks, and mock-audit notes.
- A `status.md` updated to reflect the current in-flight milestone and phase.
- An archived milestone set on completion — task plan, log, and test matrix moved together under `archive/YYYY-MM-DD/` via `docs archive --cascade`.

## When to invoke

Triggers on "create a milestone", "start M1", "next milestone", "begin implementation", "advance the project". Use after [`project-foundation`](https://github.com/ArtRichards/project-foundation) has set up the docs tree.

For end-to-end autonomous milestone execution (planning + implementation + review + simplify in one go), use [`ship-milestone`](https://github.com/ArtRichards/ship-milestone) instead — it spawns fresh sub-agents per step and conducts the whole milestone.

## Install

Install this skill through the Agent Playbook Suite marketplace plugin. The
repository root `README.md` carries the Codex and Claude Code marketplace
commands. This directory is the portable skill payload for hosts that support
direct skill-directory installs.

## Dependencies

- [`docs-cli`](https://github.com/ArtRichards/docs-cli) — always use the newest release; required for `Lifecycle:`, `docs new --body-from`, and atomic multi-file `docs touch <file>...`. The skill calls `docs new`, `docs touch`, `docs index`, `docs check`, and `docs archive --cascade` throughout. Install with `pip install --upgrade docs-cli`.
- Companion skills (recommended): [`project-foundation`](https://github.com/ArtRichards/project-foundation) (run first), [`ship-milestone`](https://github.com/ArtRichards/ship-milestone), [`sync-and-commit`](https://github.com/ArtRichards/sync-and-commit) (called at phase/step boundaries), [`simplify`](https://github.com/ArtRichards/simplify) (Phase 10).

## Convention

Follows the docs-cli convention: each Markdown file is self-describing via a metadata block under the H1; milestone task plan, impl log, and test matrix are linked with `Related: pairs-with` so cascade archive can move them together. See [`docs-cli`'s convention spec](https://github.com/ArtRichards/docs-cli/blob/main/docs/convention.md).

## License

MIT — see [`LICENSE`](LICENSE).
