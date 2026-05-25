# sync-and-commit

A Claude Code skill that verifies work is consistent, complete, and accurate; syncs the docs tree to reality; commits; and pushes (when on a feature branch with a remote).

End-of-step wrap-up. Reads `CLAUDE.md` for project conventions, walks a technical-verification checklist (code completeness, tests, lint, type safety, accuracy vs. spec), updates the docs tree, reviews the diff, and produces a single phase-scoped commit. Never bypasses git hooks; never pushes to `main`.

## When to invoke

- Automatically — called by [`ship-milestone`](https://github.com/ArtRichards/ship-milestone) at every step boundary.
- Manually — at the end of any TDD phase, or whenever the operator wants to wrap up in-flight work.

## Install

```sh
git clone https://github.com/ArtRichards/sync-and-commit \
  ~/.claude/skills/sync-and-commit
```

Claude Code auto-discovers skills in `~/.claude/skills/`. Restart Claude Code (or open a new session) and the skill is available.

## Dependencies

- A `CLAUDE.md` at the project root (recommended) describing the docs-tree location, build/test/quality commands, commit convention, and branch convention. Fallback: the skill inspects `git log --oneline`, `package.json` / `pyproject.toml` / `Makefile`.
- [`docs-cli`](https://github.com/ArtRichards/docs-cli) **v1.2.0 (M7+)** — optional. If the project uses docs-cli, the skill runs `docs touch`, `docs index`, and `docs check --stale 14` to keep the docs tree in lockstep. If not, the docs-side checks are skipped.

## Guarantees

- Never uses `--no-verify` or bypasses git hooks (unless the operator explicitly asks).
- Never pushes to `main` or any shared branch — only feature/milestone branches.
- Never blanket-adds with `git add -A` unless `CLAUDE.md` says otherwise.

## License

MIT — see [`LICENSE`](LICENSE).
