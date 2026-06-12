# sync-and-commit

An agent workflow skill that verifies work is consistent, complete, accurate,
and risk-gated; syncs the docs tree to reality; commits; and pushes when on a
safe feature branch with a remote.

End-of-step wrap-up. Reads `CLAUDE.md`, `AGENTS.md`, or equivalent project
context for conventions, walks technical verification and risk-aware quality
checks, updates the docs tree, reviews the diff, and produces a single
phase-scoped commit. Never bypasses git hooks; never pushes to `main`.

## When to invoke

- Automatically — called by [`ship-milestone`](https://github.com/ArtRichards/ship-milestone) at every step boundary.
- Manually — at the end of any TDD phase, or whenever the operator wants to wrap up in-flight work.

## Install

Install this skill through the Agent Playbook Suite marketplace plugin. The
repository root `README.md` carries the Codex and Claude Code marketplace
commands. This directory is the portable skill payload for hosts that support
direct skill-directory installs.

## Dependencies

- A `CLAUDE.md`, `AGENTS.md`, or equivalent project context at the project root (recommended) describing the docs-tree location, build/test/quality commands, risk gates, commit convention, and branch convention. Fallback: the skill inspects `git log --oneline`, `package.json` / `pyproject.toml` / `Makefile`.
- [`docs-cli`](https://github.com/ArtRichards/docs-cli) — optional, but use the newest release when present. If the project uses docs-cli, the skill runs `docs touch`, `docs index`, and `docs check --stale 14` to keep the docs tree in lockstep. If not, the docs-side checks are skipped.

## Guarantees

- Never uses `--no-verify` or bypasses git hooks (unless the operator explicitly asks).
- Never pushes to `main` or any shared branch — only feature/milestone branches.
- Never blanket-adds with `git add -A` unless project context says otherwise.
- Fails closed on selected visible product tests or explicit non-product
  checks, configured build/lint/type gates, missing Standard/High
  contract-test evidence needed to judge the gate, unexplained mocks, and
  unapproved skipped High-risk deep gates selected for the milestone.

## License

MIT — see [`LICENSE`](LICENSE).
