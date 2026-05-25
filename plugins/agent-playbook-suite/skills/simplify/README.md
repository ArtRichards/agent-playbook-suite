# simplify

A Claude Code skill for post-implementation simplification — TDD Phase 10 (Quality, Docs, Refactor).

Reduces code complexity while preserving behavior: replaces clever code with obvious code, removes abstraction layers that don't earn their keep, collapses needless helpers, favors linear execution. Anchors to the most recent commit as the known-good baseline, simplifies, then proves behavior is preserved by re-running the full suite and quality gate.

## When to invoke

Manually invoked (e.g. `/simplify`) once a milestone's implementation is complete and the test suite is green. Also called automatically by [`ship-milestone`](https://github.com/ArtRichards/ship-milestone) at Step 3.

## Hard constraints

- Does not change public behavior.
- Does not add generic architecture or new abstractions (unless they remove more complexity than they add).
- Does not rewrite working code just to make it look different.
- If nothing genuinely simplifies, makes no changes.

## Install

```sh
git clone https://github.com/ArtRichards/simplify \
  ~/.claude/skills/simplify
```

Claude Code auto-discovers skills in `~/.claude/skills/`. Restart Claude Code (or open a new session) and the skill is available.

## Dependencies

None required. The skill reads `CLAUDE.md` (if present) to discover the project's implementation-log convention; otherwise it skips the log entry rather than inventing one.

Pairs with [`sync-and-commit`](https://github.com/ArtRichards/sync-and-commit) (run after `/simplify` to commit and push the simplifications).

## License

MIT — see [`LICENSE`](LICENSE).
