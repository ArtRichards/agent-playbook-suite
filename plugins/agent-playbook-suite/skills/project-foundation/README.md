# project-foundation

A Claude Code skill that bootstraps a new project's foundation — charter, scope, architecture, milestones, definition-of-ready — as a docs-managed Markdown tree.

Use it when starting a project (or a sub-project) that needs front-half planning before any implementation. The skill walks through a ten-phase interactive wizard, authoring every artifact via [`docs new`](https://github.com/ArtRichards/docs-cli) so the metadata, lifecycle, and inter-doc graph are correct by construction. At the end you have a foundation log, charter, scope, architecture sketch, milestone plan, risk register, decision log, definition-of-ready, and (if missing) a `CLAUDE.md` that ties the docs tree to the skill ecosystem that will drive the project.

## What it produces

- A `docs/specs/` (or `./specs/`) tree, each file with `Lifecycle:` / `Role:` / `Related:` metadata.
- A foundation log capturing the history of decisions made during bootstrap.
- A charter (`Role: reference`), scope (`Role: spec`), architecture sketch (`Role: sketch`), milestone plan (`Role: spec`), risk register (`Role: reference`), decision log (`Role: log`), and a definition-of-ready (`Role: reference`).
- A `.docs.toml` configured with the project's name, vocabulary extensions, and `[exclude]` rules.
- A `CLAUDE.md` (scaffolded fresh, or extended via `CLAUDE-additions.md`) describing the docs tree, the skill ecosystem, the 10-phase TDD methodology, and the branch-stack convention used by [`ship-milestone`](https://github.com/ArtRichards/ship-milestone).

## When to invoke

Triggers on "start a project", "create a foundation", "scope a new project", "draft a charter", "plan a project from scratch".

After this skill completes, hand off to [`create-milestones`](https://github.com/ArtRichards/create-milestones) to begin implementation, milestone by milestone.

## Install

```sh
git clone https://github.com/ArtRichards/project-foundation \
  ~/.claude/skills/project-foundation
```

Claude Code auto-discovers skills in `~/.claude/skills/`. Restart Claude Code (or open a new session) and the skill is available.

## Dependencies

- [`docs-cli`](https://github.com/ArtRichards/docs-cli) **v1.2.0 (M7+)** — required. The skill calls `docs new`, `docs index`, `docs touch`, and `docs check` throughout. Install with `pip install docs-cli`.
- Companion skills (recommended): [`create-milestones`](https://github.com/ArtRichards/create-milestones), [`ship-milestone`](https://github.com/ArtRichards/ship-milestone), [`sync-and-commit`](https://github.com/ArtRichards/sync-and-commit), [`simplify`](https://github.com/ArtRichards/simplify).

## Convention

The skill follows the docs-cli convention: each Markdown file is self-describing via a metadata block under the H1; the INDEX is derived, never hand-maintained. See [`docs-cli`'s convention spec](https://github.com/ArtRichards/docs-cli/blob/main/docs/convention.md) for the full rules.

## License

MIT — see [`LICENSE`](LICENSE).
