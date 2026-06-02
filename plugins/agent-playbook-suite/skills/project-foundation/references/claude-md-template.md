# CLAUDE.md template

Drop this content at the repo root as `CLAUDE.md` or adapt the same
sections for `AGENTS.md` (alongside `README.md`). The wizard
substitutes `<placeholder>` slots from artifacts already authored in
Phases 0-7.

Anything between `<...>` in the template body is a **derived
slot** — the wizard substitutes it from foundation artifacts at
render time. The full slot list is in the
[Filling the derived slots](#filling-the-derived-slots) table
at the bottom of this file.

**Exception — literal placeholders in Branch conventions.** The
`<slug>` token in the `<slug>/phases-1-4` etc. branch names is
a **literal documentation placeholder** that ships in the
rendered CLAUDE.md unchanged. It represents "your milestone's
slug" to readers and matches how `ship-milestone` documents its
own branch shape. Do not substitute it at render time.

---

```markdown
# CLAUDE.md

## What this project is

<one-paragraph project summary, derived from charter.md's
"What we're building" section>

## Documentation tree

This project uses [`docs-cli`](https://github.com/ArtRichards/docs-cli)
to manage planning, milestone, and spec docs as a
self-describing Markdown tree.

- Docs root: `<docs-root-path>` (marked by `.docs.toml`)
- Active artifacts: charter, scope, architecture, milestone-plan,
  test-strategy, env-and-tooling, definition-of-ready, …
- Archive subtree: `<docs-root-path>/archive/<YYYY-MM-DD>/`
- Single source of truth for progress: `<docs-root-path>/status.md`
- Auto-generated machine view: `<docs-root-path>/INDEX.md`
  (never hand-edit)

**Always use `docs` CLI verbs** for metadata, lifecycle, archive,
or index actions — never hand-edit metadata blocks, the
`<!-- docs:generated -->` block in `INDEX.md`, or files into
`archive/`. See the `docs` skill for the verb table.

The controlled-vocab field is `Lifecycle:` (not `Status:`) as of
docs-cli M7+. A free-form `Status: <prose>` body line is allowed
but the skills prefer `Lifecycle: blocked` +
`Related: blocked-by: <doc>` for blocked states — queryable, not
prose.

## Skill ecosystem

This project is set up to use:

- **`docs`** ([docs-cli](https://github.com/ArtRichards/docs-cli)) —
  the convention enforcer; required by every skill below.
- **`project-foundation`** — front-half planning (charter →
  Definition of Ready). Use when scoping a new sub-project or
  major feature inside this repo.
- **`create-milestones`** — milestone-level TDD work. Use to
  create, advance, or complete one milestone interactively.
- **`ship-milestone`** — autonomous end-to-end milestone driver.
  Use to run a milestone through all 10 TDD phases unattended
  (`/ship-milestone <id>` or `/ship-milestone next milestone`).

## TDD methodology

Milestones follow a risk-aware 10-phase TDD cycle:

1. Define Contract
2. Write Tests (RED)
3. Create Data/Fixtures
4. Run Tests (RED Baseline)
5. Update Base Interfaces
6. Implement Offline/Core Path
7. Update Tool/Wrapper Layer
8. Run Tests (GREEN)
9. Integrate / Accept / Dogfood
10. Quality, Docs, Refactor

Canonical reference: the `create-milestones` skill's
`tdd-phases.md`.

## Quality gates and test adequacy

This project uses risk-aware agentic TDD. Visible tests drive
implementation, but visible tests are not sufficient evidence of
intent.

Risk levels:
- Lite:
- Standard:
- High:

Fast PR gate:
```sh
<derived from test-strategy.md fast PR gate commands>
```

Deep/nightly/release gate:
```sh
<derived from test-strategy.md deep/nightly/release gate commands or "not configured">
```

Hidden/generalization tests:
- The implementation agent must not inspect hidden cases.
- Visible tests drive implementation.
- Hidden tests, mutation, property/stateful, fuzzing, and benchmarks test adequacy and generalization.

Mock policy:
- Do not add or expand mocks without justification.
- Prefer at least one real-path test for each mocked boundary.

When in doubt:
- Stop on unresolved behavior intent.
- Do not weaken tests to pass.
- Do not special-case visible fixtures, literals, or test-only branches.

## Branch conventions

When using `ship-milestone`, milestone work lands on a stacked
branch set per milestone:

- `<slug>/milestone-setup` (only if the milestone's task plan
  doesn't exist yet)
- `<slug>/phases-1-4`
- `<slug>/phases-5-10`
- `<slug>/simplify`

Nothing merges to `main` without operator review. The branches
stack; the operator reviews and merges.

For interactive `create-milestones` use, branch conventions are
the project's choice (often `<slug>` as a single branch).

## Build, test, quality commands

<derived from env-and-tooling.md — runtimes, build, test, lint,
typecheck commands; quality-gate one-liner if applicable>

## Commit conventions

<derived from existing repo conventions (recent git log) or
defaulted to: concise, imperative, present-tense, scoped per
phase when working under ship-milestone>

## Where to read next

- [Charter](<docs-root-path>/charter.md) — problem, target user,
  success metric.
- [Status](<docs-root-path>/status.md) — current milestone and
  phase.
- [Architecture](<docs-root-path>/architecture.md) — shape,
  modules, data flow.
- [Milestone Plan](<docs-root-path>/milestone-plan.md) — the
  full milestone sequence.
- [INDEX](<docs-root-path>/INDEX.md) — machine view of every
  doc, generated.
```

## Detecting an existing CLAUDE.md

If a `CLAUDE.md` already exists at the repo root, **do not
overwrite**. Read it and check for these four fixed sections by
content match:

| Section | Detection heuristic |
|---|---|
| Documentation tree | substring match on `.docs.toml` or "docs root" or "docs-managed" |
| Skill ecosystem | substring match on at least two of `project-foundation`, `create-milestones`, `ship-milestone`, `docs-cli` |
| TDD methodology | substring match on "10-phase" or "Define Contract" + "Implement Online" |
| Quality gates and test adequacy | substring match on "risk-aware" or "hidden/generalization" or "mock policy" |
| Branch conventions | substring match on `phases-1-4` or `<slug>/phases` |

**If all four sections are already present**, skip writing
`CLAUDE-additions.md` entirely — just note "CLAUDE.md already
covers the skill ecosystem" in `foundation-log.md`. The existing
CLAUDE.md is good as-is.

**If one or more sections are missing**, write the proposed
additions to `CLAUDE-additions.md` next to `CLAUDE.md` (not into
CLAUDE.md itself). The file's body uses this shape:

```markdown
# Proposed additions to CLAUDE.md

This file was generated by the `project-foundation` skill during
the foundation work for <project-slug>. Review and merge into
`CLAUDE.md` selectively — keep what fits, discard what doesn't.

## (missing section 1)

<rendered template section, with placeholders filled>

## (missing section 2)

<rendered template section, with placeholders filled>

…
```

The user reviews `CLAUDE-additions.md` and merges into
`CLAUDE.md` by hand or with Edit calls. The skill never edits
`CLAUDE.md` directly when it pre-existed.

## Filling the derived slots

| Slot | Source |
|---|---|
| `<project summary>` | First paragraph of `charter.md`'s "What we're building" section. |
| `<docs-root-path>` | The path resolved at Bootstrap Step 1, relative to the repo root (e.g. `docs/specs` or `specs`). |
| `<project-slug>` | The kebab-case slug from `[project] name` in `.docs.toml`. |
| Build/test/quality commands | The `Build commands`, `Test commands`, and any quality-gate command sections from `env-and-tooling.md`. |
| Quality gate sections | The risk levels, fast PR gate, deep/nightly/release gate, hidden-test policy, mock policy, and human approval triggers from `test-strategy.md`. |
| Commit conventions | If recent `git log --oneline -20` shows a consistent style, summarise it in one line. Otherwise default to: "concise, imperative, present-tense; one commit per TDD phase when working under `ship-milestone`." |
