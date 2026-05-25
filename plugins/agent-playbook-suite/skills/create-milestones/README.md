# create-milestones

A Claude Code skill that creates, advances, and completes milestones for a project whose foundation work is done.

Drives the 10-phase TDD methodology (Define Contract → Write Tests RED → Create Fixtures → Run Tests RED → Update Interfaces → Implement Core → Update Wrappers → Run Tests GREEN → Integrate → Quality/Docs/Refactor). Authors a `<slug>.md` task plan + `<slug>-impl.md` implementation log pair per milestone via [`docs new`](https://github.com/ArtRichards/docs-cli), and atomically archives the pair with `docs archive --cascade` on completion.

## What it produces

- A milestone task-plan doc (`Role: spec`) capturing the 10-phase plan, decisions, deliverables, success criteria, and per-phase checklist.
- A milestone implementation log (`Role: log`, paired via `Related: pairs-with`) updated phase by phase as work progresses.
- A `status.md` updated to reflect the current in-flight milestone and phase.
- An archived milestone pair on completion — both files moved together under `archive/YYYY-MM-DD/` via `docs archive --cascade`.

## When to invoke

Triggers on "create a milestone", "start M1", "next milestone", "begin implementation", "advance the project". Use after [`project-foundation`](https://github.com/ArtRichards/project-foundation) has set up the docs tree.

For end-to-end autonomous milestone execution (planning + implementation + review + simplify in one go), use [`ship-milestone`](https://github.com/ArtRichards/ship-milestone) instead — it spawns fresh sub-agents per step and conducts the whole milestone.

## Install

```sh
git clone https://github.com/ArtRichards/create-milestones \
  ~/.claude/skills/create-milestones
```

Claude Code auto-discovers skills in `~/.claude/skills/`. Restart Claude Code (or open a new session) and the skill is available.

## Dependencies

- [`docs-cli`](https://github.com/ArtRichards/docs-cli) **v1.2.0 (M7+)** — required. The skill calls `docs new`, `docs touch`, `docs index`, `docs check`, and `docs archive --cascade` throughout. Install with `pip install docs-cli`.
- Companion skills (recommended): [`project-foundation`](https://github.com/ArtRichards/project-foundation) (run first), [`ship-milestone`](https://github.com/ArtRichards/ship-milestone), [`sync-and-commit`](https://github.com/ArtRichards/sync-and-commit) (called at phase/step boundaries), [`simplify`](https://github.com/ArtRichards/simplify) (Phase 10).

## Convention

Follows the docs-cli convention: each Markdown file is self-describing via a metadata block under the H1; milestone task plan and impl log are linked with `Related: pairs-with`. See [`docs-cli`'s convention spec](https://github.com/ArtRichards/docs-cli/blob/main/docs/convention.md).

## License

MIT — see [`LICENSE`](LICENSE).
