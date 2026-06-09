# Automated Software Development Workflow Guide

Lifecycle: active
Role: guide
Project: agent-playbook-suite
Updated: 2026-06-08

This guide explains a public workflow built from five workflow skills and the
supporting `docs` skill, distributed as one suite plugin for Codex and Claude
Code, plus one supporting CLI:

- [`project-foundation`](https://github.com/ArtRichards/project-foundation)
- [`create-milestones`](https://github.com/ArtRichards/create-milestones)
- [`ship-milestone`](https://github.com/ArtRichards/ship-milestone)
- [`sync-and-commit`](https://github.com/ArtRichards/sync-and-commit)
- [`simplify`](https://github.com/ArtRichards/simplify)
- [`docs-cli`](https://github.com/ArtRichards/docs-cli)

All six are public. The five skills are MIT-licensed and together form an opinionated way to run a software project, or a broad feature-release effort, with an AI coding agent as the main build engine.

## What problem this solves

Long-running agent projects usually break down in three places:

- the agent loses track of the real project state
- the docs drift away from the code
- restarting after a few days costs too much time

This workflow solves that by keeping project state on disk instead of in chat history.

The project charter, scope, architecture, milestone plans, implementation logs, status, and agent instructions are all written into a structured Markdown tree. A fresh agent can then rebuild context from files, not from a transcript.

That is the core idea: state lives on disk, not in chat.

## The parts and how they fit

[`docs-cli`](https://github.com/ArtRichards/docs-cli) is the substrate. It is an agent-native tool built to make structured project documentation easier and safer for agents to maintain. It manages a prescriptive Markdown tree with metadata such as `Lifecycle:`, `Role:`, project name, and typed document relationships. It can validate the tree, rebuild `INDEX.md`, move docs safely into the archive subtree, rewrite `Related:` references on rename, query the tree, and adopt existing Markdown directories into the convention.

That matters because, in this workflow, documentation is not side paperwork. It is the working memory of the project. `docs-cli` reduces the burden on the agent by programmatically maintaining document pointers, validating relationships, and preserving the shape of the docs tree. In practice, that lowers the chance of broken references, stale status pages, hand-edited metadata drift, and other avoidable documentation mistakes while the agent is recording work in progress and planned work.

The convention is portable without the CLI, but the workflow assumes you use the CLI because that is what makes the process safe for repeated agent use.

The five skills sit on top of that:

- `project-foundation` sets up the project front-half: charter, scope, architecture, milestone plan, definition of ready, status docs, and agent context (`CLAUDE.md`, `AGENTS.md`, or equivalent)
- `create-milestones` creates and advances a milestone interactively through a fixed 10-phase TDD workflow
- `ship-milestone` runs a milestone more autonomously by spawning fresh agents for planning, implementation, review, and simplification
- `sync-and-commit` verifies the work, syncs the docs tree, reviews the diff, and commits safely
- `simplify` handles the final cleanup pass by removing unnecessary complexity without changing behavior

The workflow is sequential:

`project-foundation` -> `create-milestones` or `ship-milestone` -> `sync-and-commit` -> `simplify`

## What a user needs to do

First, install the published `docs-cli` package from PyPI:

```bash
python3 -m pip install --upgrade docs-cli
docs --version
```

Then install the Agent Playbook Suite plugin from this repository's marketplace
catalog. Treat [`README.md`](README.md) as the canonical public install guide
and keep this overview as a summary.

For Codex:

```bash
codex plugin marketplace add ArtRichards/agent-playbook-suite --ref main
codex plugin add agent-playbook-suite@agent-playbook-suite
```

For Claude Code:

```bash
claude plugin marketplace add ArtRichards/agent-playbook-suite
claude plugin install agent-playbook-suite@agent-playbook-suite
```

The repository contains both marketplace catalogs:

- `.agents/plugins/marketplace.json` for Codex
- `.claude-plugin/marketplace.json` for Claude Code

Both catalogs point at the same suite plugin under
`plugins/agent-playbook-suite/`, which contains the shared `skills/` payload.

Gemini CLI and OpenCode do not consume these Codex or Claude marketplace
manifests directly. The suite still remains portable because the packaged skill
payload lives in `plugins/agent-playbook-suite/skills/`, and the individual
workflow skill repositories remain public for manual direct-skill installs when
an agent does not support these marketplace formats.

Update Codex by refreshing the marketplace:

```bash
codex plugin marketplace upgrade agent-playbook-suite
```

Update Claude Code by refreshing the marketplace and plugin:

```bash
claude plugin marketplace update agent-playbook-suite
claude plugin update agent-playbook-suite@agent-playbook-suite
```

`docs-cli` itself is lightweight: Python 3.11+, published on PyPI, and has no runtime dependencies. The `docs` console command is the tool the other skills rely on.

The workflow is still portable outside Codex and Claude Code. In Gemini CLI,
OpenCode, and similar agent shells, the marketplace support and auto-discovery
behavior differ, but the operating model still works if the agent can read the
same repo, project instruction file, docs tree, and commands.

Always use the newest PyPI `docs-cli` release; recent releases add `Lifecycle:`
metadata, agent-friendly authoring via `docs new --body-from`, and the newer
adoption-workflow helpers. The suite plugin includes the `docs` skill, and that
bundled skill is generated from `docs-cli` itself and kept in lockstep with the
newest release, so the skill instructions always match the `docs` command they
call. There is no separate older version floor to track. Run
`python3 -m pip install --upgrade docs-cli` to stay current.

## What gets created

When you start a project with `project-foundation`, the system creates the working context the agent will rely on later:

- a docs root such as `docs/specs/` or `specs/`
- a `.docs.toml` file that marks and configures the tree
- a generated `INDEX.md`
- always-on living docs such as `status.md`, `foundation-log.md`, and `risks.md`
- charter
- scope and constraints
- stakeholder and interface map
- option comparison
- architecture sketch
- decision log
- milestone plan
- environment and tooling checklist
- data plan
- test strategy
- documentation plan
- definition of ready
- `CLAUDE.md`, `AGENTS.md`, or a matching companion additions file

When you start delivery work, each milestone gets its own milestone doc,
implementation log, and test matrix; quality logs or generated report companions
are added where the risk level calls for them. Status docs are updated as phases
move forward. New milestones can be added later as the project evolves. They are
expected to inherit the overall project goals, constraints, and architecture
context rather than starting from scratch each time.

## How the workflow works

### 1. Set up the project once

Run `project-foundation` once at the start of a project. In practice, that project is often a broad feature release, delivery effort, or sub-project inside a larger repo, not necessarily the entire lifetime of the product. The agent asks structured questions and writes the answers into the docs tree as it goes.

This is not just planning paperwork. It creates the durable operating context for future sessions and future agents, and it also prepares the agent context file that the later skills expect.

### 2. Break work into technical milestones

In this system, the project is the broad release-sized effort. The milestones inside it are the smaller technical slices.

Good milestones tend to be narrow enough that one focused agent run can finish them with clear tests, docs, and exit criteria. That makes them easier to review, easier to resume, and easier to fit into a bounded usage window.

The milestone plan is not frozen on day one. As new changes, follow-up work, or release units appear, `create-milestones` is expected to be used again to add new milestones that stay aligned with the original project context and goals.

### 3. Run a milestone through the 10 phases

Use `create-milestones` for interactive operator-driven work, or `ship-milestone` for more autonomous execution.

`create-milestones` is not just for the first pass. It is also the normal way to introduce new units of work as the release continues.

For each milestone, `create-milestones` creates a milestone task-plan doc, a
paired implementation log, and a paired test matrix. The milestone moves through
the TDD phases in those artifacts, then the artifact set is archived together
with `docs archive --cascade` when the work is complete.

Every milestone follows the same TDD-shaped phases:

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

Each phase has an explicit exit condition and a log entry. The status file reflects the active milestone and current phase at all times.

### Quality model

The suite uses a risk-aware quality model around those phases. A milestone
records the behavior contract, visible red tests, hidden/generalization
strategy, adequacy checks, risk level, and mock audit notes before the work is
treated as complete.

Risk gates scale by impact. Lite work can rely on the contract, visible tests,
configured lint/type/build checks, docs validation, and ordinary review.
Standard work adds hidden/generalization smoke, property or stateful checks
where applicable, and explicit mock review. High-risk work adds stronger gates
such as mutation, security/schema/migration, benchmark, rollback, and
fresh-eyes review checks, with operator approval after the RED baseline.

The point is not more process for its own sake. The artifact trail should show
what was tested, what was intentionally not tested, and why the selected gate
matches the risk.

### 4. Use fresh agents when autonomy matters

`ship-milestone` is the most distinctive part of the suite. It acts as a conductor, not the implementer.

It can spawn fresh agents for:

- milestone setup
- planning
- implementation
- fresh-eyes review
- simplification

In practice it works across a small branch stack per milestone:

- `<slug>/milestone-setup`
- `<slug>/phases-1-4`
- `<slug>/phases-5-10`
- `<slug>/simplify`

That separation matters. A review agent that did not write the code is less likely to rationalize it. A fresh planning agent has to reconstruct the project from the artifacts on disk. That is the point. The conductor resolves the milestone, keeps the working tree clean, triages real open questions, and leaves the implementation work to fresh sub-agents.

### 5. Sync code, docs, and git state

At each step boundary, `sync-and-commit` verifies the implementation against the milestone, runs the quality gate, updates the docs, reviews the diff, and commits safely. It is part of the delivery loop, not an afterthought. In the suite, it is the git-safety layer: do not bypass hooks, do not push to `main`, and do not treat docs sync as optional cleanup.

### 6. Simplify before closing the milestone

The final pass uses `simplify` to make the code more obvious without changing behavior. It anchors to the last known-good implementation state, simplifies, reruns the selected tests and configured quality gate, and leaves the code alone if nothing truly simplifies.

## A small example

Imagine a project called "support dashboard release." That project is the broad feature-release effort. Inside it, the work is broken into several technical milestones:

- `M1`: define auth/session contracts and add login test scaffolding
- `M2`: implement session handling and get auth tests green
- `M3`: add ticket-list read models, fixtures, and list rendering tests
- `M4`: implement ticket-list loading, filtering, and wrapper validation
- `M5`: integrate the real ticket source, update docs, and simplify

That is more typical of how the workflow is meant to be used. The project holds the broad release goal; the milestones are the narrow technical slices with clear tests and clean handoff points. Each one gets its own milestone doc, implementation log, status updates, and archive trail.

If a later change introduces export requirements or audit logging, you would add `M6` or `M7` using `create-milestones`, based on the same project context, instead of opening a separate ad hoc workflow.

## What the human operator still does

The agent does most of the writing, implementation, and bookkeeping. The human still:

- answers project setup questions
- approves tradeoffs and architecture decisions
- chooses milestone boundaries
- adds new milestones as the release evolves
- resolves ambiguous product decisions
- reviews operator decision points surfaced by the conductor

This is not "press one button and disappear." It is a structured human-plus-agent workflow.

## Cost of adoption

There is some adoption cost:

- you need to accept the docs-cli document convention
- you need to maintain useful project context (`CLAUDE.md`, `AGENTS.md`, or equivalent)
- you need to work in milestone-sized slices
- you need to treat docs as part of the build system

If that sounds too rigid, this workflow is probably not a fit. If you want resumability, auditability, and easier agent handoff, that structure is exactly the value.

## Good use cases

This workflow fits best for:

- major feature releases
- greenfield products
- internal tools
- teams that want a Markdown docs tree they can query and validate programmatically
- multi-milestone side projects
- codebases that pause and resume often
- teams that want agent work to be inspectable

It is a poor fit for one-off scripts, tiny fixes, or teams that do not want process around documentation.

## Model And Budget Note

The workflow is built for terminal-first coding agents, including Codex and Claude Code, when the model can follow multi-step plans, run tools, inspect diffs, and keep docs synchronized with implementation.

Budget planning depends on the agent, model, plan, codebase size, test cost, and how much autonomy you allow. The practical planning unit is the milestone: keep milestones small enough that a fresh agent can understand the docs, make the change, run the quality gate, and leave a reviewable diff.

## The short version

Install `docs-cli` from PyPI, then install the Agent Playbook Suite plugin from the Codex or Claude marketplace catalog in this repository. Run `project-foundation` once for the broad project or feature-release effort. Break delivery into small technical milestones. Add new milestones with `create-milestones` as the release evolves. Use `create-milestones` or `ship-milestone` to drive active milestones through the 10 phases. Use `sync-and-commit` to keep code and docs aligned. Use `simplify` at the end.

The result is a project that can be resumed by a fresh agent from artifacts on disk instead of reconstructed from chat history.
