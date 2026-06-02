# Skill suite briefing — for overview + blog post generation

Lifecycle: active
Role: notes
Project: agent-playbook-suite
Updated: 2026-06-02

## What this is

Five workflow skills plus the `docs` skill, published 2026-05-25 under
[ArtRichards](https://github.com/ArtRichards), that together form an
opinionated workflow for running a software project — from blank slate to
shipped milestones. The suite is distributed as one plugin marketplace package
for Codex and Claude Code. Gemini CLI and OpenCode can still use the packaged
skill payload directly where they support skill directories. The suite builds
on [docs-cli](https://github.com/ArtRichards/docs-cli), which provides a
controlled-vocabulary convention for prescriptive Markdown documentation trees.

The five skills:

| Repo | Role in the workflow |
|---|---|
| [project-foundation](https://github.com/ArtRichards/project-foundation) | Bootstrap a new project's front-half: charter, scope, architecture, milestones, definition-of-ready. Sets up the docs tree and agent context (`CLAUDE.md`, `AGENTS.md`, or equivalent). Run once per project. |
| [create-milestones](https://github.com/ArtRichards/create-milestones) | Create / advance / complete milestones using a 10-phase TDD methodology. Authors milestone+impl-log pairs; atomically archives them on completion. Interactive: operator-driven. |
| [ship-milestone](https://github.com/ArtRichards/ship-milestone) | Autonomous milestone driver. A lightweight conductor spawns fresh Opus sub-agents per step (plan → implement → fresh-eyes review → simplify), commits each step to its own branch, surfaces only operator-decision points. |
| [sync-and-commit](https://github.com/ArtRichards/sync-and-commit) | End-of-step wrap-up. Verify the work, sync the docs tree to reality, commit, and push (when safe). Reads project context, runs the project quality gate. Never bypasses git hooks; never pushes to `main`. |
| [simplify](https://github.com/ArtRichards/simplify) | Post-implementation simplify mode for TDD Phase 10. Reduces complexity while preserving behavior — replaces clever with obvious, drops unearned abstractions. Makes no changes if nothing genuinely simplifies. |

License: MIT on all five. Authored by `Art Richards <art@bitholdersinc.com>`.

## The problem they solve

Running Claude Code as a serious build partner — not just a chat assistant — exposes three recurring pain points:

1. **State drift.** As context grows, the model loses track of what was decided, what shipped, what's in flight. Conversations end, and the project's "true state" lives only in a transcript.
2. **Doc rot.** Hand-maintained `INDEX.md`, status doc, and milestone plan drift from reality the moment implementation starts.
3. **Restart cost.** Picking up a project days later (or handing it to another agent) means re-reading sprawling chats or guessing from `git log`.

The suite's bet: if every decision, plan, milestone, and phase result is written to a **self-describing Markdown tree** as it happens — using a tool that validates lifecycle/role/relationships — then the project's state lives on disk, not in chat history. Any fresh agent (or human) can pick up from artifacts alone.

## The underlying convention (docs-cli)

The suite is built on top of [docs-cli](https://github.com/ArtRichards/docs-cli), a small Python CLI published to PyPI as `docs-cli` that manages Markdown trees with this convention:

```markdown
# Title

Lifecycle: active
Role: spec
Project: my-project
Updated: 2026-05-25

Related:
- pairs-with: convention.md
- implements: charter.md

## …
```

- `Lifecycle:` and `Role:` are controlled vocabularies (extensible per-project, additive only).
- `Related:` is a typed-edge graph: `pairs-with`, `child-of`/`parent-of`, `implements`, `spec-of`, `supersedes`/`superseded-by`, `blocked-by`, `decision`, `references`.
- `INDEX.md` is **derived** from the metadata, never hand-maintained.
- `docs check` validates the tree; `docs archive --cascade` walks the graph one hop and atomically archives related files together.

The suite skills never have to think about indexes, broken cross-references, or lifecycle drift — they call `docs new`, `docs touch`, `docs index`, `docs check`, `docs archive --cascade` and the tree stays correct.

## The 10-phase TDD methodology

`create-milestones` and `ship-milestone` drive every milestone through these phases:

0. (implicit) Milestone created with goal/scope/exit-criteria
1. Define Contract
2. Write Tests (RED — must fail)
3. Create Data/Fixtures
4. Run Tests (RED Baseline)
5. Update Base Interfaces
6. Implement Offline/Core Path
7. Update Tool/Wrapper Layer
8. Run Tests (GREEN)
9. Integrate / Accept / Dogfood
10. Quality / Docs / Refactor (where `/simplify` lives)

The phases aren't novel TDD — what's novel is that each phase has an explicit doc touchpoint, an explicit exit criterion, and an explicit log entry. The milestone task plan and its impl log are written first and updated as work progresses; status.md reflects the current in-flight phase at all times.

## The conductor + fresh-sub-agent pattern (ship-milestone)

The most distinctive piece. `ship-milestone` doesn't itself implement code — it's a **conductor** that:

1. Spawns a **milestone-creation agent** (only if the milestone's task plan, implementation log, or test matrix doesn't exist yet) on branch `<slug>/milestone-setup`.
2. Spawns a **planning agent** for phases 1-4 on branch `<slug>/phases-1-4` — fresh context, builds understanding from artifacts on disk only.
3. Spawns an **implementation agent** for phases 1-4 — separately fresh.
4. Spawns a **fresh-eyes review agent** for the diff — independent, did not build the code.
5. Repeats 2-4 for phases 5-10 on branch `<slug>/phases-5-10`.
6. Spawns a sole **simplify agent** on `<slug>/simplify`.

Every sub-agent runs the same-instance consistency audit (see `references/consistency-check.md` in the ship-milestone repo) before returning. The conductor's job is triage and operator-routing, not synthesis. Each sub-agent's clean context is the point: it can't rationalize the code it wrote because it didn't write it; it must rebuild understanding from the milestone doc, the log, the specs, and the diff.

## How they compose

A typical project lifecycle:

```
project-foundation    → set up the docs tree, charter, scope, milestones, agent context (run once)
  ↓
create-milestones     → operator-driven, walks one milestone interactively
  ↓ OR
ship-milestone        → autonomous; uses create-milestones' process, sync-and-commit, simplify
  ↓
sync-and-commit       → called at every phase/step boundary (manually or by ship-milestone)
  ↓
simplify              → Phase 10, behavior-preserving cleanup
```

The project agent context is load-bearing: every sub-agent reads `CLAUDE.md`, `AGENTS.md`, or equivalent instructions to bootstrap context. `project-foundation` scaffolds the relevant file if missing (or generates a matching additions file if one exists), with sections for the docs-tree location, build/test/quality commands, commit convention, branch-stack convention, and the skill ecosystem.

## Audience and positioning

These are not generic skills. They target a specific user: someone running a terminal-first coding agent as a long-running build partner on real projects, willing to adopt an opinionated convention to make sessions reproducible, agents interchangeable, and project state inspectable. The investment is real (learn the docs-cli convention, structure the project agent context, accept the 10-phase TDD shape) — the return is that any fresh session, on any machine, can pick up a project mid-flight by reading the docs tree.

## Install pattern

Install the runtime CLI first:

```sh
python3 -m pip install --upgrade docs-cli
```

Codex and Claude Code install the suite as one plugin from this repository's
marketplace catalogs:

```sh
codex plugin marketplace add ArtRichards/agent-playbook-suite --ref main
codex plugin add agent-playbook-suite@agent-playbook-suite

claude plugin marketplace add ArtRichards/agent-playbook-suite
claude plugin install agent-playbook-suite@agent-playbook-suite
```

The marketplace plugin packages the six public skills together: `docs`,
`project-foundation`, `create-milestones`, `ship-milestone`, `sync-and-commit`,
and `simplify`. Gemini CLI and OpenCode do not consume these marketplace
manifests directly, but the shared skill payload under
`plugins/agent-playbook-suite/skills/` remains portable for direct skill
directory installs.

## Suggested blog-post angles

1. **"Five skills, one workflow"** — a tour of the suite, ending with the conductor pattern as the punchline.
2. **"State lives on disk, not in chat"** — the philosophical bet, with docs-cli as the substrate and the skills as the workflow.
3. **"Fresh context as a feature"** — why ship-milestone spawns new agents instead of one long-running one; the cost of rationalization and the value of independent review.
4. **"From charter to ship"** — walk one hypothetical project through `project-foundation` → `ship-milestone M1` → archived milestone, showing what each skill produces.

## Pointers for deeper reading

Each repo's `README.md` has the user-facing pitch. Substance lives in:
- `SKILL.md` — the trigger, invariants, and orchestration the model reads
- `references/` (where present) — full playbooks, templates, agent prompts

The most read-worthy single files:
- `project-foundation/references/foundation-playbook.md` (10-phase wizard)
- `project-foundation/references/claude-md-template.md` (the shared context template; adapt the same sections for `AGENTS.md`)
- `create-milestones/references/tdd-phases.md` (the 10-phase canonical reference)
- `ship-milestone/references/agent-prompts.md` (the 5 sub-agent templates)
- `docs-cli/docs/convention.md` (the metadata convention everything builds on)
