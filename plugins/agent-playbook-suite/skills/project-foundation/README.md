# project-foundation

An agent workflow skill that bootstraps a new project's foundation — charter,
scope, architecture, milestones, risk-aware validation strategy,
definition-of-ready, and agent context — as a docs-managed Markdown tree.

Use it when starting a project (or a sub-project) that needs front-half
planning before any implementation. The skill walks through a ten-phase
interactive wizard, authoring every artifact via
[`docs new`](https://github.com/ArtRichards/docs-cli) so the metadata,
lifecycle, and inter-doc graph are correct by construction. At the end you have
a foundation log, charter, scope, architecture sketch, milestone plan,
risk-aware test strategy, risk register, decision log, definition-of-ready, and
agent context files (`CLAUDE.md`, `AGENTS.md`, or additions) that tie the docs
tree to the skill ecosystem that will drive the project.

## What it produces

- A `docs/specs/` (or `./specs/`) tree, each file with `Lifecycle:` / `Role:` / `Related:` metadata.
- A foundation log capturing the history of decisions made during bootstrap.
- A charter, scope, architecture sketch, milestone plan, risk-aware test strategy, risk register, decision log, and definition-of-ready.
- A `.docs.toml` configured with the project's name, vocabulary extensions, and `[exclude]` rules.
- A `CLAUDE.md` / `AGENTS.md` context file (scaffolded fresh, or extended via additions files) describing the docs tree, the skill ecosystem, the risk-aware 10-phase TDD methodology, quality gates, and the branch-stack convention used by [`ship-milestone`](https://github.com/ArtRichards/ship-milestone).

## When to invoke

Triggers on "start a project", "create a foundation", "scope a new project", "draft a charter", "plan a project from scratch".

After this skill completes, hand off to [`create-milestones`](https://github.com/ArtRichards/create-milestones) to begin implementation, milestone by milestone.

## Install

Install this skill through the Agent Playbook Suite marketplace plugin. The
repository root `README.md` carries the Codex and Claude Code marketplace
commands. This directory is the portable skill payload for hosts that support
direct skill-directory installs.

## Dependencies

- [`docs-cli`](https://github.com/ArtRichards/docs-cli) — always use the newest release; required for `Lifecycle:`, `docs new --body-from`, tree-wide `[exclude]` rules, and atomic multi-file `docs touch <file>...`. The skill calls `docs new`, `docs index`, `docs touch`, and `docs check` throughout. Install with `pip install --upgrade docs-cli`.
- Companion skills (recommended): [`create-milestones`](https://github.com/ArtRichards/create-milestones), [`ship-milestone`](https://github.com/ArtRichards/ship-milestone), [`sync-and-commit`](https://github.com/ArtRichards/sync-and-commit), [`simplify`](https://github.com/ArtRichards/simplify).

## Convention

The skill follows the docs-cli convention: each Markdown file is self-describing via a metadata block under the H1; the INDEX is derived, never hand-maintained. See [`docs-cli`'s convention spec](https://github.com/ArtRichards/docs-cli/blob/main/docs/convention.md) for the full rules.

## License

MIT — see [`LICENSE`](LICENSE).
