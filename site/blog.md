---
layout: page
title: State lives on disk, not in chat
permalink: /blog/
---

I keep running into the same problem with coding agents: they can write the code, but they are much worse at remembering the project.

This is especially obvious after the third or fourth serious session. One session decided the API. Another changed the test strategy. A third got the implementation green but left the docs in a halfway state. A week later, a new agent has to reconstruct the whole thing from chat transcripts and git history.

I do not think the right answer is "use a bigger context window." Bigger context helps, but it does not turn a chat transcript into a reliable project database.

The answer I like here is much more boring: write the state down as small Markdown records in the repository, validate them with a CLI, and teach the agent to use that CLI instead of treating chat history as memory.

That is the idea behind [docs-cli](https://github.com/ArtRichards/docs-cli) and the five Agent Skills built on top of it: [project-foundation](https://github.com/ArtRichards/project-foundation), [create-milestones](https://github.com/ArtRichards/create-milestones), [ship-milestone](https://github.com/ArtRichards/ship-milestone), [sync-and-commit](https://github.com/ArtRichards/sync-and-commit), and [simplify](https://github.com/ArtRichards/simplify).

This is not a replacement for git, tests, or review. It is a way to make agent work resumable.

In this article:

- why chat is the wrong place for project state
- how docs-cli turns Markdown into a small record system
- how the five skills turn that into a delivery workflow
- why the fresh-agent pattern is the most interesting part
- where I think the costs are

## The real problem is state drift

LLM coding tools are good enough now that the bottleneck often moves from "can it produce code?" to "can it keep a multi-day project coherent?"

The hard parts are mundane:

- What did we decide yesterday?
- Which milestone is active?
- Which tests were supposed to be red?
- Which docs are authoritative and which are historical?
- Is this implementation log paired with the right milestone?
- Did the index update after records moved?
- Can a new agent resume without reading a 70,000-token chat?

If the answer lives only in the chat, it is fragile. Chat is a working surface, not a database. It is too long, too linear, too easy to lose, and too hard for a fresh agent to query.

The more useful pattern is to keep chat disposable and put project state in the repository.

## Markdown as records

The smallest useful unit in docs-cli is a Markdown file with a metadata block:

```markdown
# <Title>

Lifecycle: active
Role: spec
Project: <project>
Updated: YYYY-MM-DD

Related:
- implements: <charter>
- pairs-with: <implementation-log>

## <First section>

...
```

That looks boring, which is the point. It is readable in a terminal, editable in any editor, reviewable in git, and structured enough for a tool to check.

docs-cli treats each managed Markdown file as a record with a lifecycle, a role, a project, an update date, and typed relationships. It derives the index from those records. It can create new records, archive them, move them, rewrite relationships, list them, validate them, and migrate existing Markdown trees into the convention.

The convention is deliberately not a static site generator, a wiki, or a project management system. Git owns history. The CLI owns lifecycle and navigation.

That boundary matters. Agents are very willing to "just edit the index" or "just move this to archive" unless the project gives them a better surface. The better surface is a small set of verbs: create, archive, move, touch, list, check, index, and migrate.

The important part is not the command syntax. It is that the command encodes the invariant: lifecycle, physical location, related records, and the generated index all move together.

## The edge cases are the evidence

The docs-cli project is a useful case study because the tool dogfoods the workflow it is meant to support. Its design did not appear fully formed. It grew by finding small documentation failures and turning them into rules.

The generated index started as a convenience, then became a contract. When index generation hit an edge case around marker handling, the fix was not "be more careful next time." It became a regression test and a stricter parser rule.

The mutating verbs exist because hand-editing metadata is exactly what agents should not be trusted to do repeatedly. Creating a record, archiving a completed record, moving a record, and bumping its update date are all small operations. They are also exactly the operations that create drift if every agent improvises them.

Validation and listing are where the convention becomes enforceable. `docs check` gives CI-usable exit codes. `docs list --json` gives agents a query surface. Malformed records should become reportable findings, not crashes.

Migration is dry-run by default. That is the right default for an inference-heavy operation. A foreign tree gets a complete plan first: one decision per record, inferred metadata, confidence, archive moves, and ambiguities. `--apply` comes later.

The bundled skill adds the missing behavioral layer. When an agent works in a managed tree, it should use the CLI instead of hand-maintaining the convention.

Packaging matters too. The public PyPI distribution is `docs-cli`, while the executable command remains `docs`. The skill ships with the package, so a user can install the CLI from PyPI and then materialize the bundled skill on the host with `docs install-skill`.

A trial across 25 real-world Markdown trees, 501 files total, produced the most useful design correction. `Status:` looked like the obvious name for a controlled lifecycle field, but existing docs often use status as free-form prose. The convention changed to `Lifecycle:` for the controlled value and preserved prose status as migrated metadata.

The same trial pushed the migration planner toward medium-confidence inference, broader role detection, project-name normalization, and a wider core role vocabulary. The dogfood result against sanitized real-tree fixtures was 88 percent high-or-medium confidence.

That is the kind of history I want in agent tooling. Not a grand architecture claim. A sequence of small failures, turned into commands, validations, and docs that future agents must read.

## The five skills make it a workflow

On its own, docs-cli gives you the substrate. The five skills turn that substrate into a coding-agent workflow, with Claude Code as the original target.

`project-foundation` runs once near the start. It creates the project front-half: charter, scope, architecture, milestone plan, living status, definition of ready, and the project instructions Claude Code will later read.

`create-milestones` is the operator-driven delivery loop. It creates a milestone record and paired implementation log, then walks the work through a fixed ten-phase TDD sequence.

`ship-milestone` is the autonomous version. Its interesting choice is that it acts as a conductor, not as the implementer. It resolves the milestone, keeps the working tree clean, and delegates planning, implementation, review, and simplification to fresh sub-agents.

`sync-and-commit` is the step boundary. It verifies the work, updates the docs tree to match reality, reviews the diff, commits, and pushes when safe. It does not bypass git hooks and it does not push to `main`.

`simplify` is the final cleanup pass. It reduces complexity while preserving behavior. If nothing genuinely simplifies, it should make no changes.

The skills are not five independent conveniences. They are one opinionated pipeline:

```text
project-foundation
  -> create-milestones or ship-milestone
  -> sync-and-commit
  -> simplify
```

The payoff is that every important project fact has a durable artifact.

## The ten phases are there for handoff

The milestone loop is a TDD loop with explicit doc touchpoints:

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

There is nothing magic about those phase names. The value is that each phase has an exit condition and a log entry.

For a human developer, that may look heavy. For an agent workflow, it buys a lot. A fresh session does not have to infer whether tests were supposed to be failing. It can read the milestone record and implementation log. It can see that Phase 2 wrote the tests, Phase 4 captured the red baseline, Phase 8 reached green, and Phase 10 updated the documentation state.

That is also why the docs tree is not "documentation after the fact." It is the control plane for the work.

## Fresh context as a feature

The most opinionated piece is `ship-milestone`.

Most agent workflows try to preserve one large context as long as possible. `ship-milestone` goes the other way. It uses a lightweight conductor and fresh sub-agents:

- one agent sets up the milestone
- one agent plans the early phases
- another implements from fresh context
- a separate agent performs fresh-eyes review
- the pattern repeats for later phases
- a final pass simplifies the result

The reason is simple: a fresh agent cannot rely on the story it told itself while writing the code. It has to reconstruct the project from artifacts on disk. If those artifacts are insufficient, that is a real defect in the workflow.

I think this is the strongest idea in the suite. Fresh context is usually treated as a limitation of LLM systems. Here it becomes a review primitive.

This also matches how I want to use coding agents in practice. I do not want one agent to hold the entire story in its head forever. I want one agent to produce an artifact, another agent to evaluate it from first principles, and a third pass to simplify the result once the behavior is locked down.

That is much closer to how useful human review works.

## What this costs

The cost is real.

You have to accept a documentation convention. You have to maintain a useful project instruction file. You have to work in milestone-sized slices. You have to treat status records and implementation logs as part of the build, not as optional housekeeping.

This is a poor fit for one-off scripts and tiny bug fixes. It is also a poor fit if you want the agent to improvise freely and keep all rationale in chat.

It is a good fit when the restart cost is high: greenfield projects, multi-milestone features, internal tools, paused side projects, and codebases where a different agent may need to resume the work next week.

The tradeoff is structure for resumability.

## What I would watch

The risk with any process like this is ceremony creep. If the docs become a second product to maintain, the workflow has failed.

The test I would use is simple: can a fresh agent resume a milestone faster because these artifacts exist? If yes, the ceremony is paying rent. If no, delete or simplify the pieces that are not helping.

The other thing I would watch is how often humans have to resolve ambiguity. Some human decision points are good. Product tradeoffs, architecture direction, and release scope should not be silently guessed by an agent. But if the same kind of low-level ambiguity appears again and again, that is probably a missing command, a missing validation rule, or a missing convention.

## How to try it

Install the CLI:

```sh
python -m pip install --upgrade docs-cli
docs install-skill
```

Then install the five workflow skills for Claude Code:

```sh
for skill in project-foundation create-milestones ship-milestone sync-and-commit simplify; do
  git clone "https://github.com/ArtRichards/$skill" "$HOME/.claude/skills/$skill"
done
```

For Codex, Gemini CLI, and OpenCode, use `agent-skill-installation.md` in this
suite as the source of truth. It lists each agent's skill directories and keeps
the workflow skills as updateable public GitHub checkouts.

For an existing repo, the practical first step is not to automate shipping. Start smaller: run migration in dry-run mode against an existing Markdown tree, inspect the ambiguities, then validate the result before applying changes. Decide whether the convention makes the tree easier to reason about. If it does, apply it and then bring in the skills.

For a new project, start with `project-foundation`, then choose whether you want the interactive loop (`create-milestones`) or the autonomous conductor (`ship-milestone`).

## Summary

If you use Claude Code for serious multi-session development, the chat transcript is the wrong place for project state.

Put the state on disk. Make the files self-describing. Generate the index. Validate the tree. Pair milestone plans with implementation logs. Make every phase leave a trail a fresh agent can read.

Then clone the five repos and install `docs-cli` from PyPI. The point is not more documentation. The point is making agent work restartable.

## Related Hacker News discussions

These are useful adjacent discussions, not source citations for the claims above:

- [How I program with LLMs](https://news.ycombinator.com/item?id=42617645) - a practical thread on where LLMs help experienced programmers and where review discipline still matters.
- [Embracing the parallel coding agent lifestyle](https://news.ycombinator.com/item?id=45489884) - directly relevant to the conductor / fresh-agent pattern and the cost of reviewing multiple agent outputs.
- [Ask HN: Senior software engineers, how do you use Claude Code?](https://news.ycombinator.com/item?id=46573860) - a broad sample of real Claude Code workflows and tradeoffs.
- [Ask HN: Do you have any evidence that agentic coding works?](https://news.ycombinator.com/item?id=46691243) - useful skepticism around what counts as evidence for agentic coding.
- [Launch HN: Nia (YC S25) - Give better context to coding agents](https://news.ycombinator.com/item?id=46194828) - related to the idea that coding agents need explicit project context, not just more tokens.
- [Show HN: Project management system for Claude Code](https://news.ycombinator.com/item?id=44960594) - adjacent to the missing project-management layer that this workflow is trying to make file-backed and auditable.
- [The 70% problem: Hard truths about AI-assisted coding](https://news.ycombinator.com/item?id=42336553) - a skeptical counterweight on why generated code often needs substantial human review.
- [AI-Assisted Coding Killed My Joy of Programming](https://news.ycombinator.com/item?id=46108704) - useful as a reminder that not every developer wants the same relationship with agentic tools.
