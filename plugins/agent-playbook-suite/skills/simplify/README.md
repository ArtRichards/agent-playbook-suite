# simplify

An agent workflow skill for post-implementation simplification — TDD Phase 10
(Quality, Docs, Refactor).

Reduces code complexity while preserving behavior and test adequacy: replaces
clever code with obvious code, removes abstraction layers that don't earn their
keep, collapses needless helpers, favors linear execution. Anchors to the most
recent commit as accepted prior work, simplifies, then checks for regressions
with the same selected gates. Still-applicable results may be reused; checks
affected by the change run again.

## When to invoke

Manually invoked (e.g. `/simplify`) once a milestone's implementation is complete and the selected product tests plus configured explicit checks are green. Also called automatically by [`ship-milestone`](https://github.com/ArtRichards/ship-milestone) at Step 3.

## Hard constraints

- Does not change public behavior.
- Does not add generic architecture or new abstractions (unless they remove more complexity than they add).
- Does not rewrite working code just to make it look different.
- Does not reduce meaningful protection from selected visible, property/stateful,
  hidden-hook, mutation, fuzz, benchmark, security, schema, fixture, or real-path
  checks without explicit logged approval.
- Does not replace real-path tests with mocks.
- If nothing genuinely simplifies, makes no changes.

## Install

Install this skill through the Agent Playbook Suite marketplace plugin. The
repository root `README.md` carries the Codex and Claude Code marketplace
commands. This directory is the portable skill payload for hosts that support
direct skill-directory installs.

## Dependencies

None required. The skill reads `CLAUDE.md`, `AGENTS.md`, or equivalent context
if present to discover the project's implementation-log convention, risk
level, and gate set; otherwise it skips the log entry rather than inventing
one.

Pairs with [`sync-and-commit`](https://github.com/ArtRichards/sync-and-commit) (run after `/simplify` to commit and push the simplifications).

## License

MIT — see [`LICENSE`](LICENSE).
