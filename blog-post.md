# How I run Claude Code on multi-week projects

Lifecycle: active
Role: notes
Project: agent-playbook-suite
Updated: 2026-08-16

[Agent Playbook Suite](https://github.com/ArtRichards/agent-playbook-suite) is one plugin for Claude Code and Codex that gives the agent a repeatable delivery process. [`docs-cli`](https://github.com/ArtRichards/docs-cli) is the small Python CLI underneath it that keeps project state on disk instead of in chat. I use them together so that a fresh agent can pick up a multi-week project mid-stream and keep going without me re-explaining anything.

This post shows how to use the suite, how to use docs-cli, and why it works.

## Try it

```sh
python3 -m pip install --upgrade docs-cli
claude plugin marketplace add ArtRichards/agent-playbook-suite
claude plugin install agent-playbook-suite@agent-playbook-suite
```

(Codex: `codex plugin marketplace add ArtRichards/agent-playbook-suite --ref main`, then `codex plugin add agent-playbook-suite@agent-playbook-suite`.)

New project: run [`project-foundation`](https://github.com/ArtRichards/project-foundation)
and drive the first milestone with
[`create-milestones`](https://github.com/ArtRichards/create-milestones).
Existing repo: start with `docs migrate` on your Markdown folder and judge the
convention on its own before adopting the workflow. The rest of this post
explains what you just installed.

## Using the playbook suite

The suite provides eight workflow skills plus the supporting `docs` skill.
Seven form the ordinary delivery path;
[`explore`](https://github.com/ArtRichards/agent-playbook-suite/tree/main/plugins/agent-playbook-suite/skills/explore)
is an optional cross-cutting capability for the cases where the technical route
is genuinely uncertain.

**1. Start the project once with `project-foundation`.** The agent inspects the repo, then walks you through charter, scope, architecture, milestone plan, and test strategy — proposing answers from what it found, so you correct a draft instead of filling out a form. Risk levels are proposed with reasoning and default to Standard; the agent has to ask before it can call anything High. The output is a docs tree plus your `CLAUDE.md` or `AGENTS.md`, and two living logs: one for engineering follow-ups, one for feedback and ideas.

**2. Pin the use cases with [`use-cases`](https://github.com/ArtRichards/agent-playbook-suite/tree/main/plugins/agent-playbook-suite/skills/use-cases).** Runs automatically when foundation finishes. You and the agent work out how a user would actually use the thing — it brings suggestions rather than demanding answers — and the primary use cases land in a `use-cases.md` doc. Tests focus on these first. It is optional, but skipping it means your tests lose their main anchor.

**3. Keep identity separate from order with [`manage-milestone-tracker`](https://github.com/ArtRichards/agent-playbook-suite/tree/main/plugins/agent-playbook-suite/skills/manage-milestone-tracker).** Milestones use stable semantic slugs such as `fetch-and-parse`; `milestone-plan.md` stores explicit integer order, state, and dependencies. Inserting work changes an order value, not filenames or branch history, and both delivery skills derive `next` from the same eligible tracker rows.

**4. Run each milestone with `create-milestones` or [`ship-milestone`](https://github.com/ArtRichards/ship-milestone).** Same ten TDD phases either way: contract, failing tests, fixtures, RED baseline, implementation, GREEN, integration, cleanup. `create-milestones` is interactive — you confirm each phase. `ship-milestone` is autonomous — a conductor spawns fresh sub-agents to create missing milestone artifacts, plan, and implement on a stacked set of branches. At the end of Steps 0–2 it attempts isolated reviews from one Claude-family and one GPT-family agent, and nothing merges to `main` without you.

**5. Wrap every step with [`sync-and-commit`](https://github.com/ArtRichards/sync-and-commit).** It verifies the work against the milestone and tracker, syncs the docs tree to reality, and commits. It never bypasses git hooks and never pushes `main`. If the docs disagree with the code, the commit does not happen.

**6. Close each milestone with [`simplify`](https://github.com/ArtRichards/simplify).** A behavior-preserving cleanup pass. If nothing genuinely simplifies, it changes nothing. High-risk code simplification gets one fresh review or explicit operator approval. Autonomous shipping commits a clean pre-archive checkpoint, previews the exact archive scope, moves only the completed milestone artifacts, marks the tracker row complete, and checks every metadata and body link. That checkpoint makes an interrupted closeout recoverable without replaying the archive.

**Cross-cutting: use `explore` only for clear technical uncertainty.** It can
compare architecture routes during foundation, select an implementation route
after a milestone contract is stable, or recover after evidence invalidates a
selected route. It is not a mandatory stage and does not trigger merely because
the code is unfamiliar. Automatic handoff requires a named downstream consumer
and a hard signal such as an unverified acceptance-critical assumption, no
applicable pattern, unresolved mechanisms, or concrete route invalidation. The
skill first maps the repository's existing patterns, then compares materially
different mechanisms against the contract and the simplest viable
implementation. It may run isolated disposable probes, but it does not write
production implementation or persist dependency or configuration changes. If
the evidence is not strong enough, it returns the exact gap or the
operator-owned decision instead of presenting a guess as a solution.

Two small conventions do a lot of the lifting. Anything that surfaces mid-flight — feedback, an idea, a deferred check — goes into the project logs, not into the tail of a milestone doc that is about to be archived. And milestone meaning lives in a semantic slug while order lives in the tracker, so inserting or rearranging work never renames activated files, branches, or history.

Operator questions follow the same discipline. When a workflow genuinely needs
your decision, it explains in plain language where the question came from, why
the answer matters now, what each option changes, and a concrete example; it
gives an evidence-backed recommendation when the project supports one. Routine,
reversible choices are still resolved without interrupting you.

## Using docs-cli

The skills write everything through `docs`, a stdlib-only Python CLI (Python 3.11+, on PyPI). Every document carries a small metadata block — lifecycle, role, project, typed links to related docs — and `INDEX.md` is generated, never hand-edited. In a dedicated project docs root, the verbs are mechanical:

```sh
docs new milestone fetch-and-parse --project demo --title "Fetch and parse"
docs touch fetch-and-parse.md --check
docs archive fetch-and-parse.md --cascade-dry-run --cascade-only 'fetch-and-parse-*'
docs archive fetch-and-parse.md --cascade-only 'fetch-and-parse-*' --reason "Milestone complete"
```

In a shared docs root, the tracker still displays the semantic slug, while the
suite uses a repository-unique project slug and qualifies the project
directory, physical artifact stem, archive scope, and branch prefix. That
prevents two projects with the same milestone name from colliding in branches
or when docs-cli places archived basenames in one dated directory.

`docs check` validates the whole tree: missing fields, broken links, lifecycle drift, stale docs. That is the gate the skills run at every step boundary.

You can use docs-cli without the suite. `docs install-skill` puts the bundled skill on any Claude Code host, and `docs migrate ./notes/` adopts an existing Markdown folder into the convention — dry-run by default, so you read the plan before anything changes. Many repos benefit from just the managed tree and the check gate; the milestone workflow is an optional layer on top.

Why a CLI instead of just telling the agent to keep Markdown tidy? Because consistency is exactly what agents are bad at across sessions. The CLI makes the invariants mechanical, so no session can drift the tree, and no agent burns tokens re-deriving the project's shape.

## Why

The failures I hit running Claude Code on long projects were never about code generation. They were about continuity. One session decides the API, the next writes tests against a different shape, and a week later a fresh agent reads an inconsistent repo and asks me a question I already answered.

The suite's bet is simple: anything that lives only in chat is lost when the session ends. So every decision, plan, phase result, and open question is forced into the docs tree as it happens. Resuming a project means reading the canonical tracker, `status.md`, and the selected milestone's plan and implementation log — not scrolling transcripts. I built docs-cli itself this way; the full milestone trail is public in [its `docs/` directory](https://github.com/ArtRichards/docs-cli/tree/main/docs).

Three more reasons the structure earns its keep:

- **Independent, cross-provider review.** At the end of Steps 0–2, `ship-milestone` freezes one decision-relevant evidence packet and independently attempts one fresh Claude-family reviewer and one fresh GPT-family reviewer. Neither sees the other's conclusions before both attempts are terminal. If one provider is unavailable, it uses exactly one reviewer from the available provider — not a second same-provider substitute. Every blocker must be fixed, disproven with contract, code, or test evidence, or routed to you as a genuine product decision. Review repeats only when objective evidence cannot close a concern, a correction materially changes the reviewed approach, or conflicting concerns need reconciliation; both reviews repeat only after a whole-step material change.
- **Tests with taste, not just tests.** Agent-written tests drift toward whatever is easiest to assert, which is usually the current implementation. The suite's quality model names this and pushes back. In Kent Beck's test-desiderata terms it wants tests that are behavioral and structure-insensitive: sensitive to changes in behavior, unmoved by refactors that keep behavior fixed. So it treats an overconstrained test — a byte-exact golden, an exhaustive snapshot, a change-detector that mirrors the code — as a defect to flag, exactly like an underconstrained one. The rule is to prefer the least constraining check that still gives real confidence, anchor those checks to the primary use cases, and never freeze incidental representation unless the exact bytes are the contract. Risk level decides how much validation runs beyond that.
- **No dead artifacts.** Every doc, log, and phase output is supposed to have a live downstream consumer — a demand-driven chain the suite calls information liveness. A milestone contract feeds the test matrix; the status file feeds the next session; a decision note feeds the reviewer. An artifact nobody reads is not neutral, it is a smell: either it is missing a consumer or it should not exist. This is the same discipline that makes a week-later handoff cheap, applied to the docs themselves.

## What it costs

- You learn a small, opinionated convention: one metadata block per file, a typed link graph, a generated index.
- You work in milestone-sized slices. Drive-by edits do not fit.
- The docs tree is part of the build. `sync-and-commit` blocks commits that diverge from it.
- Genuinely novel work can spend extra agent and probe time in `explore`; ordinary work stays on the direct delivery path.
- `ship-milestone` can add up to two initial review calls after each of Steps 0–2, plus conditional targeted or two-provider re-review calls. The plugin does not include provider accounts, credentials, or cross-provider access; dual review requires both Claude- and GPT-family mechanisms to be authorized already. With one available provider, it runs one initial review.

For a one-file fix, this is over-engineered — keep the rationale in chat and move on. It pays off when restart cost is high: greenfield builds, multi-week features, any project a different agent will resume later.

## Three examples

### A new tool, from nothing

Say you want a CLI that crawls a site and reports broken links. In an empty repo, you ask the agent to start a project; `project-foundation` picks up, looks at what exists (nothing yet), and asks the front-half questions one phase at a time: what problem, for whom, what is out of scope, what are the milestones, how will you know it works. Your answers become files:

```
docs/specs/
├── charter.md            what we're building and why
├── scope-and-constraints.md
├── architecture.md
├── milestone-plan.md     semantic slugs, explicit order/state/dependencies
├── test-strategy.md      risk levels and gates
├── use-cases.md          how a user will actually use it
├── followup-log.md       open engineering items
├── feedback-log.md       feedback and ideas, as they come up
├── status.md             current milestone and phase
└── INDEX.md              generated map of all of it
```

Then you say "start fetch-and-parse." The agent claims that tracker row, creates a milestone doc (the contract: inputs, outputs, error cases, what done means), a test matrix mapped to your use cases, and an implementation log — and walks the ten phases, recording each one as it lands.

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

The payoff case. You shelved a project mid-milestone; today you (or a different agent, or a different machine) pick it up. The session reads four files:

```
milestone-plan.md  → session-storage is active at order 200
status.md          → "Session storage. Phase 6 of 10. Last completed:
                      Phase 5, base interfaces, 2026-06-05."
session-storage.md → the contract, deliverables, and open questions
session-storage-impl.md → every phase entry and decision so far
```

That is the entire handoff. No transcript archaeology, no "let me re-explain the architecture." The agent resumes at Phase 6 knowing what was decided, what is left, and what counts as done — because every previous session was required to write that down before it could commit.

The test that matters: can a fresh session pick up your project without you explaining it? That is what this is for.

## Try it on a real project

Install the current CLI from PyPI, then install Agent Playbook Suite from this
repository's Codex or Claude Code marketplace:

```sh
python3 -m pip install --upgrade docs-cli

# Codex
codex plugin marketplace add ArtRichards/agent-playbook-suite --ref main
codex plugin add agent-playbook-suite@agent-playbook-suite

# Claude Code
claude plugin marketplace add ArtRichards/agent-playbook-suite
claude plugin install agent-playbook-suite@agent-playbook-suite
```

Pick one real, bounded milestone and see whether the next fresh session can
resume it from the repository without an explanation.
