---
layout: page
title: How I run Claude Code on multi-week projects
permalink: /blog/
---

Most failures I have hit running Claude Code as a serious build partner are not about code generation. They are about continuity.

A session decides the API. The next session writes tests against a different shape. A third session lands the implementation but forgets to update the milestone doc. Two days later a fresh agent picks up the project, reads `git log`, reads `INDEX.md`, finds them inconsistent, and asks me a question I already answered. Multiply that by a multi-week project and the agent is no longer a build partner — it is a very expensive intern with amnesia.

[Agent Playbook Suite](https://github.com/ArtRichards/agent-playbook-suite) is the workflow I use to push back against that. It is one plugin for Codex and Claude Code that bundles five workflow skills on top of [`docs-cli`](https://github.com/ArtRichards/docs-cli), a small Python CLI that manages a prescriptive Markdown tree. The skills give the agent a process; the CLI gives the process an audit trail.

This post explains what the suite is for, how I actually use it, and three concrete cases where it has earned its keep.

## What it is

Six pieces, in install order:

- [`docs-cli`](https://github.com/ArtRichards/docs-cli) — a small Python CLI (`docs new`, `docs index`, `docs check`, `docs archive --cascade`, `docs migrate`) that manages a Markdown tree as a graph of self-describing records. Published on PyPI as `docs-cli`. Stdlib-only, Python 3.11+.
- [`project-foundation`](https://github.com/ArtRichards/project-foundation) — bootstraps a new project's front-half: charter, scope, architecture, milestone plan, `CLAUDE.md`. Run once.
- [`create-milestones`](https://github.com/ArtRichards/create-milestones) — interactive milestone driver. Walks one milestone through ten TDD phases.
- [`ship-milestone`](https://github.com/ArtRichards/ship-milestone) — autonomous milestone driver. Conductor that spawns fresh sub-agents for planning, implementation, review, and simplification.
- [`sync-and-commit`](https://github.com/ArtRichards/sync-and-commit) — end-of-step wrap-up. Verifies, syncs docs to reality, commits, pushes when safe.
- [`simplify`](https://github.com/ArtRichards/simplify) — Phase 10 cleanup. Reduces complexity without changing behavior. Makes no changes if nothing genuinely simplifies.

The five skills are MIT-licensed. The whole bundle installs from one marketplace.

## What it is for

The suite is for one specific kind of work: **multi-session, multi-milestone software delivery where the project state has to survive a fresh agent picking it up.**

That includes:

- greenfield products and libraries
- internal tools you keep returning to
- large feature releases that span weeks
- side projects that pause and resume
- any codebase where you expect to hand the work to a different agent next week

It is explicitly not for one-shot scripts, single-file fixes, or work where you are happy to keep the rationale in chat.

The bet the workflow makes is simple. Anything that lives only in chat is lost the moment the session ends. So the suite forces every decision, milestone plan, phase result, and open question into a Markdown tree on disk. `docs-cli` keeps that tree internally consistent so the agent does not spend tokens re-deriving the project's shape.

## The workflow in one screen

```
project-foundation        once, at the start
  ↓
create-milestones         interactive milestone loop
  OR
ship-milestone            autonomous milestone loop
  ↓ (called from inside both)
sync-and-commit           every step boundary
  ↓ (last phase of each milestone)
simplify                  behavior-preserving cleanup
```

Every milestone moves through ten phases:

1. Define Contract
2. Write Tests (RED — must fail)
3. Create Fixtures
4. Run Tests (confirm RED baseline)
5. Update Interfaces
6. Implement Core
7. Update Wrappers
8. Run Tests (GREEN)
9. Integrate
10. Quality, Docs, Refactor

This is not novel TDD. What is novel is that each phase has an explicit log entry, an exit criterion, and a status update. The milestone task plan and its paired implementation log are written first and updated as work progresses. `status.md` reflects the current in-flight phase at all times.

That sounds heavy. For ten-minute tasks it is. For ten-day tasks it is the difference between resuming and restarting.

## How I actually use it

### Day 0: `project-foundation`

I run `project-foundation` once and answer a structured set of questions. The agent writes:

- a charter (`docs/charter.md`)
- scope and constraints
- an architecture sketch
- a milestone plan
- a definition of ready
- a living `status.md`
- a `CLAUDE.md` (or a `CLAUDE-additions.md` diff if one exists)

Every file lands with `docs new`, so the metadata block is correct without me thinking about it:

```markdown
# docs — Charter

Lifecycle: active
Role: charter
Project: docs
Updated: 2026-05-24

Related:
- pairs-with: convention.md
- pairs-with: cli.md
- pairs-with: plan.md
```

`INDEX.md` is generated. I never hand-edit it.

### Per milestone: `create-milestones` or `ship-milestone`

For milestones where I want to stay in the loop, I run `create-milestones`. It creates the milestone doc and the paired implementation log, then walks me through the ten phases. At the end of each phase the log gets an entry and the status doc moves forward.

For milestones where I want the agent to run unattended for a stretch, I run `ship-milestone`. This is the most distinctive part of the suite. `ship-milestone` does not write the code itself. It acts as a conductor and spawns fresh sub-agents per branch:

- `<slug>/milestone-setup` — milestone-creation agent (only if the task plan and log do not exist)
- `<slug>/phases-1-4` — fresh planning agent, then fresh implementation agent
- `<slug>/phases-5-10` — fresh planning agent, then fresh implementation agent
- `<slug>/simplify` — fresh simplify agent

Between each step a fresh-eyes review agent reads the diff cold. Each sub-agent runs a same-instance consistency audit before returning. The conductor's job is triage and operator routing, not synthesis.

The point of fresh contexts is not throughput. It is that a review agent that did not write the code cannot rationalize it. It has to rebuild understanding from the milestone doc, the implementation log, the specs, and the diff. If it cannot, the artifact trail is incomplete and the workflow has told me something useful.

### Per step boundary: `sync-and-commit`

`sync-and-commit` is the git-safety layer. It runs the project's quality gate, verifies the implementation against the milestone, updates the docs tree, reviews the diff, and commits. It does not bypass hooks. It does not push to `main`. If the docs tree disagrees with the code, the commit does not happen.

### End of milestone: `simplify`

`simplify` runs the Phase 10 cleanup pass. It replaces clever code with obvious code, drops abstractions that did not earn their place, and reruns the test suite. If nothing genuinely simplifies, it returns without changes. That refusal is part of the contract — the skill is not allowed to invent work.

## Three use cases, with examples

### 1. Greenfield: building a new tool

This is the case the suite was designed for. The clearest worked example is `docs-cli` itself: I used the suite to build the CLI that the suite depends on. The full artifact trail is in the public repo at [github.com/ArtRichards/docs-cli](https://github.com/ArtRichards/docs-cli).

Ten milestones, all visible:

- M1 — parser and `docs index`
- M2 — mutating verbs (`new`, `archive`, `mv`, `touch`)
- M3 — validation and query (`check`, `list`)
- M4 — migration helper (`docs migrate`)
- M5 — Claude Code skill bundle
- M6 — PyPI packaging preparation
- M7 — migration plan accuracy
- M8 — adoption workflow (agent-driveable)
- M9 — PyPI publish 1.3.0
- M10 — adoption-flow polish

Each milestone is a `docs/mN-*.md` task plan paired with a `docs/mN-*-log.md` implementation log. Every TDD phase has an entry. The milestone-completion summaries record what shipped, what was deferred, and what changed mid-flight. The mid-M6 "scope reframe" — where M6 was reframed from "publish to PyPI" to "preparation only," with the actual publish deferred to M9 — is a single-paragraph callout at the top of the milestone doc, written when the decision was made.

That callout is the thing I would have lost in chat. It is the kind of mid-project rescope that no agent could reconstruct from `git log` alone, because the commit messages describe what shipped, not what was decided not to ship.

When I pick the project back up after a week, the first three files I read are `status.md`, the current milestone doc, and the impl log. That is faster than scrolling chat for any project longer than a day.

### 2. Adoption: a non-conforming Markdown tree

The second use case is bringing an existing Markdown directory under the convention. `docs migrate` does inference — for each file it guesses a lifecycle, a role, related edges, and a confidence score — and emits a dry-run plan with a per-file table and a footer summary.

A typical adoption looks like this:

```sh
# 1. Inspect what would change
docs migrate ./notes/

# 2. Triage the ambiguous entries
docs migrate ./notes/ --summary --only ambiguous

# 3. Exclude data subdirs and re-run
echo 'fixtures/' >> ./notes/.docsignore
docs migrate ./notes/

# 4. Apply when the plan looks right
docs migrate ./notes/ --apply

# 5. Verify
docs check ./notes/
```

The migration is dry-run by default because inference on real-world Markdown is messy. The first version of the convention used `Status:` for the controlled lifecycle field. Dogfooding against existing trees made it obvious that "status" in the wild is usually free-form prose, so the convention switched to `Lifecycle:` for the controlled value and preserved any existing `Status:` content as migrated metadata. That kind of correction only comes from running the tool on inputs it did not expect.

I use this flow whenever I inherit a repo with a `docs/` or `notes/` folder and want the agent to maintain it instead of fighting it. The agent calls `docs new --body-from -` whenever it authors a new doc, so the metadata stays correct without the agent needing to remember the convention.

### 3. Multi-session handoff

The third case is the one that motivated the whole suite. A real feature release spans days. Sessions get interrupted. The agent fills its context and gets compacted. I switch machines. Sometimes I switch agents entirely — Claude Code for the planning, Codex for the implementation, whichever works that day.

Without the suite, every handoff costs something. With the suite, the handoff is reading three files:

- `status.md` — what milestone is active, what phase, what was last completed
- the current milestone doc — the contract, the deliverables, the exit criteria, open questions
- the current implementation log — every phase entry, every deviation, every decision recorded as it happened

A fresh agent can resume from those three files. I have done it. The suite is at the point where a session can hand off mid-milestone — say, between phases 4 and 5 — and the next session reconstructs intent without me typing a paragraph of context.

The test I apply: can a fresh agent resume faster because these artifacts exist? If yes, the workflow is paying rent. If no, simplify it.

## What it costs

The cost is real and I would rather state it than hide it:

- You have to learn the docs-cli convention. It is small (one metadata block per file, a typed `Related:` graph, a generated `INDEX.md`) but it is opinionated.
- You have to maintain a useful `CLAUDE.md`. The sub-agents read it to bootstrap. If it lies, they will too.
- You have to work in milestone-sized slices. Drive-by edits do not fit.
- You have to treat the docs tree as part of the build. `sync-and-commit` will block a commit that diverges from the milestone.

If any of those sound like overhead with no return, the suite is not the right tool. For a one-file bug fix it is wildly over-engineered.

## When it pays back

The workflow earns its place when restart cost is high. Greenfield projects, multi-milestone features, internal tools you keep returning to, any codebase where you expect a different agent to resume the work. The more sessions a project spans, the more the artifact trail is worth.

It also pays back when you want independent review. The conductor pattern in `ship-milestone` is the cleanest version of "fresh eyes on the diff" I have used. The review agent has nothing but the artifacts on disk and the diff. If it cannot reconstruct the change, that is a real finding — not a model issue.

## Try it

Install the runtime CLI:

```sh
python3 -m pip install --upgrade docs-cli
docs --version
```

Install the suite plugin. For Codex:

```sh
codex plugin marketplace add ArtRichards/agent-playbook-suite --ref main
codex plugin add agent-playbook-suite@agent-playbook-suite
```

For Claude Code:

```sh
claude plugin marketplace add ArtRichards/agent-playbook-suite
claude plugin install agent-playbook-suite@agent-playbook-suite
```

Gemini CLI and OpenCode do not consume these marketplace manifests directly, but the shared skill payload lives in `plugins/agent-playbook-suite/skills/` and is portable for direct skill-directory installs.

For an existing repo, start small: run `docs migrate <dir>` against an existing Markdown tree, read the dry-run plan, exclude what does not belong, apply, and verify. If the convention makes the tree easier to reason about, bring in the skills.

For a new project, start with `project-foundation`, then pick `create-milestones` for an interactive loop or `ship-milestone` for an autonomous one. Look at `docs-cli`'s own [`docs/`](https://github.com/ArtRichards/docs-cli/tree/main/docs) directory while you are at it — that is what the artifact trail looks like when the workflow has been running for ten milestones.

## Summary

- The suite is for multi-session, multi-milestone software work where state has to survive between sessions.
- Project state lives in a Markdown tree managed by `docs-cli`, not in chat history.
- `project-foundation` sets up the front-half. `create-milestones` or `ship-milestone` runs each milestone through ten TDD phases. `sync-and-commit` is the git-safety layer. `simplify` is the closing pass.
- `ship-milestone` uses fresh sub-agents on purpose — independent review is the point.
- The cost is the convention and the milestone discipline. The payoff is that any fresh agent can resume the project from the docs tree.

Install `docs-cli` from PyPI, install the suite plugin from the marketplace, and run `project-foundation` on your next greenfield project. Then judge it on whether a fresh session can pick up the work without you explaining it.
