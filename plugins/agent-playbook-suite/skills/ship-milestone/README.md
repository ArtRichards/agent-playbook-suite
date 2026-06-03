# ship-milestone

An agent workflow skill that autonomously runs a milestone through all ten
risk-aware TDD phases to completion.

A lightweight conductor spawns fresh high-capability sub-agents — milestone
creation (if needed), then per-step planning, implementation, and fresh-eyes
review — commits each step to its own branch, and finishes with `/simplify`.
Each sub-agent starts with a clean context, builds understanding from artifacts
on disk, and returns a structured report; the conductor's job is triage, not
implementation. Use Codex `gpt-5.5` with `xhigh` when available, or Claude Opus
with deep thinking on Claude Code.

## Step model

The conductor walks a milestone through four steps, each on its own branch:

1. `<slug>/milestone-setup` — milestone creation agent (only if the milestone's task plan, implementation log, or test matrix does not yet exist).
2. `<slug>/phases-1-4` — planning agent → implementation agent (contract, visible RED tests, hidden/generalization categories, test matrix) → fresh-eyes review and risk-aware RED checkpoint.
3. `<slug>/phases-5-10` — planning agent → implementation agent (implement, integrate, quality) → fresh-eyes review.
4. `<slug>/simplify` — sole simplify agent ([`/simplify`](https://github.com/ArtRichards/simplify) skill), then [`sync-and-commit`](https://github.com/ArtRichards/sync-and-commit).

Every sub-agent runs the same-instance consistency / completeness / accuracy audit (see [`references/consistency-check.md`](references/consistency-check.md)) before returning to the conductor. The conductor surfaces open questions to the operator and resumes the sub-agent with answers.

## When to invoke

Use when the operator wants a milestone built end-to-end without per-phase hand-holding. Invoke as `/ship-milestone <milestone-id>` (e.g. `/ship-milestone M2`) or `/ship-milestone next milestone`.

## Install

Install this skill through the Agent Playbook Suite marketplace plugin. The
repository root `README.md` carries the Codex and Claude Code marketplace
commands. This directory is the portable skill payload for hosts that support
direct skill-directory installs.

## Dependencies

- [`docs-cli`](https://github.com/ArtRichards/docs-cli) — always use the newest release; required for `Lifecycle:`, `docs new --body-from`, and atomic multi-file `docs touch <file>...`. Install with `pip install --upgrade docs-cli`.
- Companion skills — required:
  - [`create-milestones`](https://github.com/ArtRichards/create-milestones) (process + 10-phase TDD structure).
  - [`sync-and-commit`](https://github.com/ArtRichards/sync-and-commit) (called at every step boundary).
  - [`simplify`](https://github.com/ArtRichards/simplify) (Step 3).
- Companion skill — recommended: [`project-foundation`](https://github.com/ArtRichards/project-foundation) (run once before the first milestone).
- `CLAUDE.md`, `AGENTS.md`, or equivalent project context at the project root documenting the docs tree location, build/test/quality commands, risk gates, commit convention, and branch convention. Sub-agents read this to bootstrap context.

## License

MIT — see [`LICENSE`](LICENSE).
