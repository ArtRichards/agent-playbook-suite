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

**Exception — literal naming placeholder.** The `<slug>` token in milestone
artifact and branch naming examples is a **literal documentation placeholder**
that ships in the rendered CLAUDE.md unchanged. It represents "your
milestone's semantic slug" to readers and matches how `ship-milestone`
documents its own branch shape. Do not substitute it at render time.
`<project-branch-prefix>` is different: after verifying that the project slug
is repository-unique, render it as `<project-slug>/` whenever this repository
contains more than one suite project, and as the empty string only when one
project-wide milestone namespace is guaranteed.

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
- Project path inside that root: `<project-path>` (empty for a dedicated
  project root; otherwise a root-relative prefix ending in `/`). Docs-cli
  document operands and `Related:` targets use this prefix; Markdown body
  links between docs in this same directory stay relative.
- Active artifacts: charter, scope, architecture, milestone-plan,
  test-strategy, env-and-tooling, definition-of-ready, …
- Archive subtree: `<docs-root-path>/archive/<YYYY-MM-DD>/`
- Canonical milestone tracker:
  `<docs-root-path>/<project-path>milestone-plan.md`.
  Its exact `Order | Milestone | State | Depends on | Notes` table owns
  semantic identity, execution order and state, dependencies, and derived
  next-work selection.
- Milestone artifact filenames use a physical artifact stem: `<slug>` in a
  dedicated root, or `<project-slug>-<slug>` in a shared root, followed by
  `.md`, `-impl.md`, or `-test-matrix.md`. The tracker still displays
  `<slug>`. Project qualification is required because docs-cli flattens
  archived files to basenames; every physical stem must be root-globally
  unique.
- Narrative project summary: `<docs-root-path>/<project-path>status.md`. It
  links to the tracker but does not duplicate its rows or independently name
  schedule or next work.
- Open follow-ups and feedback:
  `<docs-root-path>/<project-path>followup-log.md` (engineering follow-ups) and
  `<docs-root-path>/<project-path>feedback-log.md`
  (operator feedback, ideas, scope thoughts). These are the single
  home for open items — milestone docs reference open entries, and
  an entry moves into the milestone that incorporates it. Each log
  embeds its own entry template.
- Auto-generated machine view: `<docs-root-path>/INDEX.md`
  (never hand-edit)

**Always use `docs` CLI verbs** for metadata, reciprocal relationships,
lifecycle, archive, or index actions — never hand-edit metadata blocks, the
`<!-- docs:generated -->` block in `INDEX.md`, or files into
`archive/`. See the `docs` skill for the verb table.

Run docs-cli from `<docs-root-path>`. Relative `FILE`, `SOURCE`, and `TARGET`
operands are interpreted from the current working directory; `--root` selects
the tree but does not rebase those operands. Within that working directory,
use the root-relative `<project-path>` prefix described above.

The controlled-vocab metadata field is `Lifecycle:` (not `Status:`).
Document lifecycle is separate from milestone execution state. The tracker
stores `planned`, `active`, `paused`, `complete`, or `cancelled`; current
blocking is derived from live `blocked-by` relationships, written with
`docs relate` so its reciprocal `blocks` edge lands too.

## Skill ecosystem

This project is set up to use:

- **`docs`** ([docs-cli](https://github.com/ArtRichards/docs-cli)) —
  the convention enforcer; required by every skill below.
- **`project-foundation`** — front-half planning (charter →
  Definition of Ready). Use when scoping a new sub-project or
  major feature inside this repo.
- **`use-cases`** — collaborative use-case exploration after
  foundation work (optional, but strongly preferred). Produces
  `use-cases.md`; milestone tests focus primarily on the primary
  use cases recorded there.
- **`explore`** — bounded technical route exploration for a clearly
  uncertain implementation or genuinely novel problem. Used selectively
  during architecture choice, stable-contract route selection, or recovery
  from an invalidated route; it does not implement production code.
- **`manage-milestone-tracker`** — direct semantic tracker operations such
  as inspect, insert, reorder, normalize, pause, resume, cancel, dependency
  changes, and deterministic next-work discovery. It does not implement or
  archive milestones.
- **`create-milestones`** — milestone-level TDD work. Use to
  create, advance, or complete one milestone interactively.
- **`ship-milestone`** — autonomous end-to-end milestone driver.
  Use to run a milestone through all 10 TDD phases unattended
  (`/ship-milestone <semantic-slug>` or
  `/ship-milestone next milestone`).

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

Product tests validate shipped behavior and may belong in default
test discovery. Non-product checks validate planning, documentation,
handoff, or workflow artifacts and must be invoked explicitly.

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

## Operator questions and decisions

Investigate the repository, project docs, prior answers, and applicable policy
before asking the operator for input. Resolve objective questions from that
evidence when the workflow permits it; do not add approval gates for routine,
reversible work.

When a question, clarification, approval, or decision is necessary, explain in
plain language where it came from, why it is needed now, and what the answer
changes in practice. Include a concise project-grounded example, or a clearly
labeled hypothetical when the project has no suitable example. Recommend the
choice supported by project evidence and explain why; if the evidence does not
support a preference, say there is no strong recommendation. Keep the
explanation proportional to the decision rather than forcing a fixed template
or length.

## Branch conventions

When using `ship-milestone`, milestone work lands on a stacked
branch set per milestone:

- `<project-branch-prefix><slug>/milestone-setup` (only if the milestone's task plan
  doesn't exist yet)
- `<project-branch-prefix><slug>/phases-1-4`
- `<project-branch-prefix><slug>/phases-5-10`
- `<project-branch-prefix><slug>/simplify`

Every suite project has a repository-unique project slug. The rendered
`<project-branch-prefix>` is mandatory when this repository has multiple suite
projects. For example,
`payments/session-storage/phases-1-4` and
`identity/session-storage/phases-1-4` stay distinct. Omit it only when the
repository guarantees a single project-wide milestone namespace.

Nothing merges to `main` without operator review. The branches
stack; the operator reviews and merges.

For interactive `create-milestones` use, branch conventions are the project's
choice, but the same collision rule applies (often
`<project-branch-prefix><slug>` as a single branch).

## Build, test, quality commands

<derived from env-and-tooling.md — runtimes, build, test, lint,
typecheck commands; quality-gate one-liner if applicable>

## Commit conventions

<derived from existing repo conventions (recent git log) or
defaulted to: concise, imperative, present-tense, scoped per
phase when working under ship-milestone>

## Where to read next

- [Charter](<docs-root-path>/<project-path>charter.md) — problem, target user,
  success metric.
- [Status](<docs-root-path>/<project-path>status.md) — narrative project
  summary.
- [Architecture](<docs-root-path>/<project-path>architecture.md) — shape,
  modules, data flow.
- [Milestone Plan](<docs-root-path>/<project-path>milestone-plan.md) — the
  canonical milestone identity, order, execution state, dependencies,
  and source for derived next work.
- [INDEX](<docs-root-path>/INDEX.md) — machine view of every
  doc, generated.
```

## Detecting existing agent context

If a `CLAUDE.md` or `AGENTS.md` already exists at the repo root,
**do not overwrite it**. Read each existing file and check for
these six fixed sections by content match:

| Section | Detection heuristic |
|---|---|
| Documentation tree | substring match on `.docs.toml` or "docs root" or "docs-managed" |
| Skill ecosystem | substring match on at least two of `project-foundation`, `create-milestones`, `ship-milestone`, `docs-cli` |
| TDD methodology | substring match on "10-phase" or "Define Contract" + "Implement Online" |
| Quality gates and test adequacy | substring match on "risk-aware" or "hidden/generalization" or "mock policy" |
| Operator questions and decisions | substring match on "operator questions" or "why it is needed now" + "recommendation" |
| Branch conventions | substring match on `phases-1-4` plus a project-prefix collision rule |

**If all six sections are already present**, skip writing the
corresponding additions file entirely — just note that the agent
context already covers the skill ecosystem in `foundation-log.md`.
The existing file is good as-is.

**If one or more sections are missing**, write the proposed
additions to `CLAUDE-additions.md` or `AGENTS-additions.md` next to
the existing context file (not into that file). The additions file
uses this shape:

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

Use `# Proposed additions to AGENTS.md` in the corresponding
`AGENTS-additions.md` file.

The user reviews the additions file and merges it into the
corresponding context file by hand or with Edit calls. The skill
never edits a pre-existing `CLAUDE.md` or `AGENTS.md` directly.

## Filling the derived slots

| Slot | Source |
|---|---|
| `<project summary>` | First paragraph of `charter.md`'s "What we're building" section. |
| `<docs-root-path>` | The path resolved at Bootstrap Step 1, relative to the repo root (e.g. `docs/specs` or `specs`). |
| `<project-path>` | The docs-root-relative directory prefix established at Bootstrap Step 1, including its trailing slash; empty for a dedicated project root (e.g. `specs/payments/` in a shared root). |
| `<project-slug>` | The repository-unique kebab-case `Project:` value resolved at Bootstrap Step 3 (`[project] name` supplies the default in a dedicated root). |
| `<project-branch-prefix>` | `<project-slug>/` when the repository has multiple suite projects; otherwise empty. Freeze it at first milestone activation. |
| Build/test/quality commands | The `Build commands`, `Test commands`, and any quality-gate command sections from `env-and-tooling.md`. |
| Quality gate sections | The risk levels, fast PR gate, deep/nightly/release gate, hidden-test policy, mock policy, and human approval triggers from `test-strategy.md`. |
| Commit conventions | If recent `git log --oneline -20` shows a consistent style, summarise it in one line. Otherwise default to: "concise, imperative, present-tense; one commit per TDD phase when working under `ship-milestone`." |
