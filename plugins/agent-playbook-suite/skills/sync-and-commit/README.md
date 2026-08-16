# sync-and-commit

An agent workflow skill that verifies work is consistent, complete, accurate,
and risk-gated; verifies semantic milestone tracker state and any already
applied archive closeout; syncs active docs; commits; and pushes when on a safe
feature branch with a remote.

End-of-step wrap-up. Reads `CLAUDE.md`, `AGENTS.md`, or equivalent project
context for conventions, walks technical verification and risk-aware quality
checks, updates the docs tree, reviews the diff, and produces a single
phase-scoped commit. Never bypasses git hooks; never pushes to `main`.
It never performs milestone archival or edits/touches an archived milestone
document.

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
- [`docs-cli`](https://github.com/ArtRichards/docs-cli) — optional for ordinary
  projects; use 2.0 or newer for suite milestone closeout. If the project uses
  docs-cli, the skill keeps active docs in lockstep and runs `docs check
  --stale 14`. If not, docs-side checks are skipped.

## Guarantees

- Never uses `--no-verify` or bypasses git hooks (unless the operator explicitly asks).
- Never pushes to `main` or any shared branch — only feature/milestone branches.
- Never blanket-adds with `git add -A` unless project context says otherwise.
- For tracked milestone work, verifies that the canonical row, semantic slug,
  concrete project path, separate path-free artifact stem, project-qualified
  companion paths, dependencies, and `<project-branch-prefix><slug>/...`
  branch root agree. Project slugs are repository-globally unique; the prefix is
  `<project>/` with multiple suite projects and empty only for one project-wide
  namespace. `status.md` remains a tracker-linked narrative, not a scheduler.
- For explicit `ship-milestone` Step 3, normal closeout requires matching JSON
  preview/apply evidence for a caller-supplied frozen
  `<project-path><artifact-stem>-*` scope, where project path is empty or a
  trailing-slash docs-root-relative prefix. Semantic slug remains tracker and
  branch identity. The skill verifies—but never derives or repairs—the
  expected path/stem mapping, identical scope in both commands, and root-global
  destination uniqueness required by docs-cli's flattened archive paths. It
  also verifies the immutable Step 2 base, clean pre-archive checkpoint SHA/tree,
  no code change after it, and tree-bound reviewer/operator approval when a
  High-risk simplify pass changed code. It then verifies the same row in
  `complete` state, resolvable paths, and a clean docs check. After a verified
  interruption that already moved the exact set,
  it instead accepts clearly labeled reconstruction from the established
  branch diff, archive witnesses, rebased links, and docs check; it never
  invents missing JSON or reruns preview/archive.
- Fails closed on selected visible product tests or explicit non-product
  checks, configured build/lint/type gates, missing Standard/High
  contract-test evidence needed to judge the gate, unexplained mocks, and
  unapproved skipped High-risk deep gates selected for the milestone.

## License

MIT — see [`LICENSE`](LICENSE).
