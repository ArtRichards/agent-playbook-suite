# ship-milestone

A Claude Code skill that autonomously runs a milestone through all ten TDD phases to completion.

A lightweight conductor spawns fresh Opus sub-agents — milestone creation (if needed), then per-step planning, implementation, and fresh-eyes review — commits each step to its own branch, and finishes with `/simplify`. Each sub-agent starts with a clean context, builds understanding from artifacts on disk, and returns a structured report; the conductor's job is triage, not implementation.

## Step model

The conductor walks a milestone through four steps, each on its own branch:

1. `<slug>/milestone-setup` — milestone creation agent (only if the milestone's task plan + log don't yet exist).
2. `<slug>/phases-1-4` — planning agent → implementation agent (RED baseline + contract tests) → fresh-eyes review.
3. `<slug>/phases-5-10` — planning agent → implementation agent (implement, integrate, quality) → fresh-eyes review.
4. `<slug>/simplify` — sole simplify agent ([`/simplify`](https://github.com/ArtRichards/simplify) skill), then [`sync-and-commit`](https://github.com/ArtRichards/sync-and-commit).

Every sub-agent runs the same-instance consistency / completeness / accuracy audit (see [`references/consistency-check.md`](references/consistency-check.md)) before returning to the conductor. The conductor surfaces open questions to the operator and resumes the sub-agent with answers.

## When to invoke

Use when the operator wants a milestone built end-to-end without per-phase hand-holding. Invoke as `/ship-milestone <milestone-id>` (e.g. `/ship-milestone M2`) or `/ship-milestone next milestone`.

## Install

```sh
git clone https://github.com/ArtRichards/ship-milestone \
  ~/.claude/skills/ship-milestone
```

Claude Code auto-discovers skills in `~/.claude/skills/`. Restart Claude Code (or open a new session) and the skill is available.

## Dependencies

- [`docs-cli`](https://github.com/ArtRichards/docs-cli) **v1.2.0 (M7+)** — required. Install with `pip install docs-cli`.
- Companion skills — required:
  - [`create-milestones`](https://github.com/ArtRichards/create-milestones) (process + 10-phase TDD structure).
  - [`sync-and-commit`](https://github.com/ArtRichards/sync-and-commit) (called at every step boundary).
  - [`simplify`](https://github.com/ArtRichards/simplify) (Step 3).
- Companion skill — recommended: [`project-foundation`](https://github.com/ArtRichards/project-foundation) (run once before the first milestone).
- A `CLAUDE.md` at the project root documenting the docs tree location, build/test/quality commands, commit convention, and branch convention. Sub-agents read this to bootstrap context.

## License

MIT — see [`LICENSE`](LICENSE).
