# How I run Claude Code on multi-week projects

Lifecycle: active
Role: notes
Project: agent-playbook-suite
Updated: 2026-06-12

[Agent Playbook Suite](https://github.com/ArtRichards/agent-playbook-suite) is one plugin for Claude Code and Codex that gives the agent a repeatable delivery process. [`docs-cli`](https://github.com/ArtRichards/docs-cli) is the small Python CLI underneath it that keeps project state on disk instead of in chat. I use them together so that a fresh agent can pick up a multi-week project mid-stream and keep going without me re-explaining anything.

This post shows how to use the suite, how to use docs-cli, and why it works.

## Try it

```sh
python3 -m pip install --upgrade docs-cli
claude plugin marketplace add ArtRichards/agent-playbook-suite
claude plugin install agent-playbook-suite@agent-playbook-suite
```

(Codex: `codex plugin marketplace add ArtRichards/agent-playbook-suite --ref main`, then `codex plugin add agent-playbook-suite@agent-playbook-suite`.)

New project: run `project-foundation` and drive the first milestone with `create-milestones`. Existing repo: start with `docs migrate` on your Markdown folder and judge the convention on its own before adopting the workflow. The rest of this post explains what you just installed.

## Using the playbook suite

The suite is six skills that run in a fixed order.

**1. Start the project once with `project-foundation`.** The agent inspects the repo, then walks you through charter, scope, architecture, milestone plan, and test strategy — proposing answers from what it found, so you correct a draft instead of filling out a form. Risk levels are proposed with reasoning and default to Standard; the agent has to ask before it can call anything High. The output is a docs tree plus your `CLAUDE.md` or `AGENTS.md`, and two living logs: one for engineering follow-ups, one for feedback and ideas.

**2. Pin the use cases with `use-cases`.** Runs automatically when foundation finishes. You and the agent work out how a user would actually use the thing — it brings suggestions rather than demanding answers — and the primary use cases land in a `use-cases.md` doc. Tests focus on these first. It is optional, but skipping it means your tests lose their main anchor.

**3. Run each milestone with `create-milestones` or `ship-milestone`.** Same ten TDD phases either way: contract, failing tests, fixtures, RED baseline, implementation, GREEN, integration, cleanup. `create-milestones` is interactive — you confirm each phase. `ship-milestone` is autonomous — a conductor spawns fresh sub-agents to plan, implement, and review on a stacked set of branches, and nothing merges to `main` without you.

**4. Wrap every step with `sync-and-commit`.** It verifies the work against the milestone, syncs the docs tree to reality, and commits. It never bypasses git hooks and never pushes `main`. If the docs disagree with the code, the commit does not happen.

**5. Close each milestone with `simplify`.** A behavior-preserving cleanup pass. If nothing genuinely simplifies, it changes nothing — the skill is not allowed to invent work.

Two small conventions do a lot of the lifting. Anything that surfaces mid-flight — feedback, an idea, a deferred check — goes into the project logs, not into the tail of a milestone doc that is about to be archived. And new work slots between milestones by id (`m5a` runs between `m5` and `m6`) instead of renumbering, so filenames, branches, and history never break.

## Using docs-cli

The skills write everything through `docs`, a stdlib-only Python CLI (Python 3.11+, on PyPI). Every document carries a small metadata block — lifecycle, role, project, typed links to related docs — and `INDEX.md` is generated, never hand-edited. The verbs are mechanical:

```sh
docs new milestone m1-fetch --project demo --title "M1 — Fetch"
docs touch m1-fetch.md --check        # bump date, reindex, validate, one step
docs archive m1-fetch.md --cascade    # milestone + log + matrix move to archive/
```

`docs check` validates the whole tree: missing fields, broken links, lifecycle drift, stale docs. That is the gate the skills run at every step boundary.

You can use docs-cli without the suite. `docs install-skill` puts the bundled skill on any Claude Code host, and `docs migrate ./notes/` adopts an existing Markdown folder into the convention — dry-run by default, so you read the plan before anything changes. Many repos benefit from just the managed tree and the check gate; the milestone workflow is an optional layer on top.

Why a CLI instead of just telling the agent to keep Markdown tidy? Because consistency is exactly what agents are bad at across sessions. The CLI makes the invariants mechanical, so no session can drift the tree, and no agent burns tokens re-deriving the project's shape.

## Why

The failures I hit running Claude Code on long projects were never about code generation. They were about continuity. One session decides the API, the next writes tests against a different shape, and a week later a fresh agent reads an inconsistent repo and asks me a question I already answered.

The suite's bet is simple: anything that lives only in chat is lost when the session ends. So every decision, plan, phase result, and open question is forced into the docs tree as it happens. Resuming a project means reading three files — `status.md`, the current milestone doc, the implementation log — not scrolling transcripts. I built docs-cli itself this way; the full milestone trail is public in [its `docs/` directory](https://github.com/ArtRichards/docs-cli/tree/main/docs).

Two more reasons the structure earns its keep:

- **Independent review.** `ship-milestone`'s review agents are spawned fresh, with nothing but the docs and the diff. A reviewer that did not write the code cannot rationalize it — and if it cannot reconstruct the change from what is on disk, that itself is a finding.
- **Tests that check intent, not implementation.** Agent-written tests drift toward whatever is easiest to assert. The suite's quality model pushes the other way: tests focus on the primary use cases, prefer the least constraining check that gives real confidence, and never freeze incidental representation — no byte-exact goldens unless the bytes are the contract. Risk level decides how much validation runs beyond that.

## What it costs

- You learn a small, opinionated convention: one metadata block per file, a typed link graph, a generated index.
- You work in milestone-sized slices. Drive-by edits do not fit.
- The docs tree is part of the build. `sync-and-commit` blocks commits that diverge from it.

For a one-file fix, this is over-engineered — keep the rationale in chat and move on. It pays off when restart cost is high: greenfield builds, multi-week features, any project a different agent will resume later.

## Three examples

### A new tool, from nothing

Say you want a CLI that crawls a site and reports broken links. In an empty repo, you ask the agent to start a project; `project-foundation` picks up, looks at what exists (nothing yet), and asks the front-half questions one phase at a time: what problem, for whom, what is out of scope, what are the milestones, how will you know it works. Your answers become files:

```
docs/specs/
├── charter.md            what we're building and why
├── scope-and-constraints.md
├── architecture.md
├── milestone-plan.md     M1 fetch/parse, M2 persistence, M3 reporting
├── test-strategy.md      risk levels and gates
├── use-cases.md          how a user will actually use it
├── followup-log.md       open engineering items
├── feedback-log.md       feedback and ideas, as they come up
├── status.md             current milestone and phase
└── INDEX.md              generated map of all of it
```

Then you say "start M1." The agent creates a milestone doc (the contract: inputs, outputs, error cases, what done means), a test matrix mapped to your use cases, and an implementation log — and walks the ten phases, recording each one as it lands.

This is not hypothetical: docs-cli was built this way, and its repo keeps the whole trail — ten milestones of plans and logs in [its `docs/` directory](https://github.com/ArtRichards/docs-cli/tree/main/docs). My favorite artifact in there: mid-project, milestone M6 was reframed from "publish to PyPI" to "preparation only," and the reframe is a one-paragraph note written the moment it was decided. `git log` could never tell you that — commits record what shipped, not what you decided not to ship.

### An existing repo with a messy notes folder

You don't need a new project to start. If you have a `notes/` or `docs/` folder of accumulated Markdown, adopt it:

```sh
docs migrate ./notes/             # dry run: per-file table of guessed
                                  # role, lifecycle, and confidence
echo 'fixtures/' >> ./notes/.docsignore
docs migrate ./notes/ --apply     # stamp metadata, generate INDEX.md
docs check ./notes/               # validate the result
```

The dry run prints what it would do to every file and how confident it is, so you triage the ambiguous ones before anything changes. After `--apply`, the folder has a metadata block per file, a generated index, and a validation gate you can run in CI. The agent maintains it from then on — no suite, no milestones, just a docs tree that stays consistent.

### Coming back after a week

The payoff case. You shelved a project mid-milestone; today you (or a different agent, or a different machine) pick it up. The session reads three files:

```
status.md          → "M2 — Persistence. Phase 6 of 10. Last completed:
                      Phase 5, base interfaces, 2026-06-05."
m2-persistence.md  → the contract, deliverables, and open questions
m2-persistence-impl.md → every phase entry and decision so far
```

That is the entire handoff. No transcript archaeology, no "let me re-explain the architecture." The agent resumes at Phase 6 knowing what was decided, what is left, and what counts as done — because every previous session was required to write that down before it could commit.

The test that matters: can a fresh session pick up your project without you explaining it? That is what this is for.
