# State lives on disk, not in chat

Lifecycle: active
Role: notes
Project: agent-playbook-suite
Updated: 2026-05-25

Coding agents are good at producing changes. They are much worse at preserving project state across sessions.

**BLUF:** if you want Claude Code or another coding agent to work on serious multi-session projects, put the project state in the repository, not in the transcript. Keep decisions, milestones, status, implementation logs, and handoff notes as structured Markdown records. Validate and mutate those records with a CLI. Then teach the agent to use that CLI whenever it plans, ships, reviews, or closes work.

That is the idea behind [docs-cli](https://github.com/ArtRichards/docs-cli) and five Agent Skills built on top of it: [project-foundation](https://github.com/ArtRichards/project-foundation), [create-milestones](https://github.com/ArtRichards/create-milestones), [ship-milestone](https://github.com/ArtRichards/ship-milestone), [sync-and-commit](https://github.com/ArtRichards/sync-and-commit), and [simplify](https://github.com/ArtRichards/simplify).

This is not a replacement for git, tests, or review. It is a way to make agent work resumable.

## The problem

The common failure mode is not "the model cannot write code." It is this:

- one session decides the API
- another changes the test strategy
- a third gets the implementation green
- the docs drift
- the next agent has to infer the real state from chat, commits, and half-updated notes

Bigger context windows help, but they do not make chat a project database. Chat is linear, lossy, and hard to query. It should be a working surface, not the source of truth.

The useful pattern is to keep chat disposable and make the repository carry the state.

## Markdown as records

docs-cli treats Markdown files as small records. A managed record has a lifecycle, role, project, update date, and typed relationships:

```markdown
# <Title>

Lifecycle: active
Role: spec
Project: <project>
Updated: YYYY-MM-DD

Related:
- implements: <charter>
- pairs-with: <implementation-log>
```

This is deliberately boring. It is readable in a terminal, editable in any editor, reviewable in git, and structured enough for a tool to check.

The CLI derives the index from those records. It creates, archives, moves, touches, lists, validates, and migrates them. That matters because agents are very willing to "just edit the index" or "just move this to archive" unless the project gives them a better operation.

The operation should encode the invariant. If a completed milestone is archived, its lifecycle, physical location, related log, and generated index should move together. That should not depend on an agent remembering every convention by hand.

## Why the CLI matters

The most convincing parts of docs-cli are the small edge cases it absorbed.

Index generation became a contract, not a convenience. Marker handling, nested records, archived records, and link stability all had to be pinned down because generated navigation rots quickly when every update is ad hoc.

Validation became necessary once the convention mattered. `docs check` gives CI-usable exit codes. `docs list --json` gives agents a query surface. Malformed records become findings, not mystery failures.

Migration is dry-run by default, which is the right default for an inference-heavy operation. A foreign Markdown tree gets a plan first: inferred metadata, confidence, archive moves, and ambiguities. Applying changes comes after inspection.

The most useful correction came from trying it on real documentation trees. `Status:` looked like the obvious name for a controlled lifecycle field, but existing docs often use status as free-form prose. The convention changed to `Lifecycle:` for the controlled value and preserved prose status as migrated metadata. That is the kind of change you only get by dogfooding on messy real inputs.

The lesson is simple: if agents will rely on a convention, the convention needs commands and checks. Otherwise it is just prose the agent may or may not follow.

## The five skills

docs-cli is the substrate. The five skills turn it into a workflow.

`project-foundation` runs near the start. It creates the project front-half: charter, scope, architecture, milestone plan, living status, definition of ready, and the project instructions the agent will later read.

`create-milestones` is the operator-driven delivery loop. It creates a milestone record and paired implementation log, then walks work through a fixed ten-phase TDD sequence.

`ship-milestone` is the autonomous version. It acts as a conductor, not as the implementer. It delegates planning, implementation, review, and simplification to fresh sub-agents.

`sync-and-commit` is the step boundary. It verifies work, syncs docs to reality, reviews the diff, commits, and pushes when safe.

`simplify` is the final cleanup pass. It reduces complexity while preserving behavior, and should make no changes if nothing genuinely simplifies.

The point is not five separate conveniences. The point is that every important project fact gets a durable artifact.

## The ten phases

The milestone loop uses a TDD-shaped sequence:

1. Define Contract
2. Write Tests
3. Create Fixtures
4. Confirm RED baseline
5. Update Interfaces
6. Implement Core
7. Update Wrappers
8. Reach GREEN
9. Integrate
10. Quality, Docs, Refactor

There is nothing magic about those names. The value is that each phase has an exit condition and a log entry.

For a human, that can look heavy. For an agent workflow, it buys cheap handoff. A fresh session does not need to guess whether tests were supposed to fail. It can read what phase the project is in, what changed, what passed, and what remains.

That makes the docs tree the control plane for the work, not documentation after the fact.

## Fresh context as a feature

The most interesting idea is in `ship-milestone`.

Most agent workflows try to preserve one large context as long as possible. `ship-milestone` goes the other way. It uses fresh sub-agents for planning, implementation, review, and simplification.

That is a useful constraint. A fresh review agent cannot rely on the story the implementation agent told itself. It has to reconstruct the project from artifacts on disk. If it cannot, the artifact trail is incomplete.

This turns fresh context from a limitation into a review primitive.

That is how I want to use coding agents: one agent produces an artifact, another evaluates it from first principles, and a final pass simplifies the result once behavior is locked down.

## The cost

The cost is real.

You have to accept a documentation convention. You have to maintain useful project instructions. You have to work in milestone-sized slices. You have to treat status records and implementation logs as part of the build.

This is a poor fit for one-off scripts and tiny bug fixes. It is also a poor fit if you want the agent to improvise freely and keep rationale in chat.

It is a good fit when restart cost is high: greenfield projects, multi-milestone features, internal tools, paused side projects, and codebases where a different agent may need to resume the work next week.

The test I would use is simple: can a fresh agent resume faster because these artifacts exist? If yes, the ceremony is paying rent. If no, simplify it.

## How to try it

Install the CLI:

```sh
python3 -m pip install --upgrade docs-cli
docs --version
```

Then install the suite plugin from this repository's marketplace.

For Codex:

```sh
codex plugin marketplace add ArtRichards/agent-playbook-suite --ref main
codex plugin add agent-playbook-suite@agent-playbook-suite
```

For Claude Code:

```sh
claude plugin marketplace add ArtRichards/agent-playbook-suite
claude plugin install agent-playbook-suite@agent-playbook-suite
```

Gemini CLI and OpenCode do not consume the Codex or Claude marketplace
manifests directly, but the same skill payload is packaged in the repository
under `plugins/agent-playbook-suite/skills/`.

For an existing repo, start small. Run migration in dry-run mode against an existing Markdown tree. Inspect the ambiguities. Validate the result before applying changes. If the convention makes the tree easier to reason about, then bring in the skills.

For a new project, start with `project-foundation`, then choose the interactive loop (`create-milestones`) or the autonomous conductor (`ship-milestone`).

## Summary

If you use coding agents for serious multi-session development, the chat transcript is the wrong place for project state.

Put the state on disk. Make the files self-describing. Generate the index. Validate the tree. Pair milestone plans with implementation logs. Make every phase leave a trail a fresh agent can read.

Then install the suite plugin and keep `docs-cli` current from PyPI. The point is not more documentation. The point is making agent work restartable.
