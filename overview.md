# Agent Playbook Suite overview

Lifecycle: active
Role: guide
Project: agent-playbook-suite
Updated: 2026-08-16

Agent Playbook Suite is a public plugin for Codex and Claude Code that keeps
long-running software work understandable across agents and sessions. It
bundles eight workflow skills plus the supporting
[`docs`](https://github.com/ArtRichards/docs-cli) skill; those workflows use
the separately installed `docs-cli` runtime.

The central idea is simple: project state belongs in versioned artifacts, not
only in chat history. The suite records scope, architecture, decisions,
milestones, tests, implementation evidence, and current state in a validated
Markdown tree. A fresh agent can reconstruct the work from the repository
instead of relying on a prior conversation.

## What it solves

Long-running agent work commonly loses continuity: the plan diverges from the
code, status becomes ambiguous, and restarting requires reconstructing context.
The suite replaces that informal memory with a small operating system for the
project:

- `docs-cli` owns document metadata, relationships, indexing, validation, and
  controlled archival.
- A canonical milestone tracker separates meaningful milestone identity from
  execution order and state.
- A fixed, risk-aware TDD loop records what was promised, tested, implemented,
  reviewed, and deferred.
- Fresh-agent boundaries reduce self-review bias during autonomous delivery.

The process is deliberately opinionated. Its value is resumability,
auditability, and safer handoffs; its cost is maintaining the artifacts as part
of the work.

## The pieces

| Skill | Responsibility |
|---|---|
| [`project-foundation`](https://github.com/ArtRichards/project-foundation) | Establish charter, scope, architecture, readiness, quality strategy, milestone plan, and agent context. |
| [`use-cases`](https://github.com/ArtRichards/agent-playbook-suite/tree/main/plugins/agent-playbook-suite/skills/use-cases) | Record how people should use the result so milestone tests start from user-visible behavior. |
| [`explore`](https://github.com/ArtRichards/agent-playbook-suite/tree/main/plugins/agent-playbook-suite/skills/explore) | Compare genuinely uncertain implementation routes using repository evidence and bounded probes. |
| [`manage-milestone-tracker`](https://github.com/ArtRichards/agent-playbook-suite/tree/main/plugins/agent-playbook-suite/skills/manage-milestone-tracker) | Insert, reorder, pause, resume, cancel, and select semantic milestones without implementing them. |
| [`create-milestones`](https://github.com/ArtRichards/create-milestones) | Materialize and advance a milestone interactively through the ten TDD phases. |
| [`ship-milestone`](https://github.com/ArtRichards/ship-milestone) | Conduct an eligible milestone autonomously through planning, implementation, review, simplification, and closeout. |
| [`sync-and-commit`](https://github.com/ArtRichards/sync-and-commit) | Verify code, docs, tracker, and git state, then commit and push safely. |
| [`simplify`](https://github.com/ArtRichards/simplify) | Reduce unnecessary complexity after behavior is green, without changing the contract. |

`explore` is optional and cross-cutting. It runs only when direct inspection or
a cheap probe leaves a named, acceptance-relevant technical decision
unresolved. It does not produce mergeable production implementation.

## Adoption and installation

Install the current CLI from PyPI:

```bash
python3 -m pip install --upgrade docs-cli
docs --version
```

Then install the suite from this repository. The canonical, current commands
are in [`README.md`](README.md).

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

Both marketplaces install the same payload under
`plugins/agent-playbook-suite/skills/`. Installation does not grant access to
either model provider. Cross-provider review works only when the host already
has authorized Claude-family and GPT-family worker mechanisms.

## How the workflow runs

Foundation creates or joins an internal docs tree, then writes the project’s
living status and logs, planning documents, canonical milestone tracker, and
root agent instructions. `use-cases` follows automatically unless the operator
skips it.

The tracker has exactly five columns:

```text
Order | Milestone | State | Depends on | Notes
```

Milestones use semantic slugs such as `session-storage`; order is a separate
integer and may change without renaming files or branches. State is one of
`planned`, `active`, `paused`, `complete`, or `cancelled`. The next milestone
is derived deterministically from order, dependencies, readiness, and live
blockers. A writer claims it as `active` before creating its first branch.
Project slugs are repository-unique, so multi-project branch prefixes remain
unambiguous even when projects reuse the same milestone slug.

`create-milestones` provides the interactive path. `ship-milestone` provides
the autonomous path, using a four-branch stack for setup, contract/RED work,
implementation/quality, and simplify/closeout. Both drive the same phases:

1. Define Contract
2. Write Tests (RED)
3. Create Data/Fixtures
4. Run Tests (RED Baseline)
5. Update Base Interfaces
6. Implement Offline/Core Path
7. Update Tool/Wrapper Layer
8. Run Tests (GREEN)
9. Integrate, Accept, and Dogfood
10. Quality, Docs, and Refactor

The shared quality model scales checks by Lite, Standard, or High risk. It
prefers semantic behavior tests over incidental representation, records hidden
or generalization coverage in the implementation log and test matrix, audits
mocks, and checks that new interfaces have a real downstream consumer. Durable
quality logs are added where the selected risk gates need them. High-risk work
adds the relevant deeper gates and explicit approvals.

At the end of completed autonomous Steps 0–2, the conductor freezes one
evidence packet and attempts one isolated Claude-family review and one isolated
GPT-family review. If a provider is unavailable, one reviewer is sufficient;
the workflow never substitutes a second reviewer from the same family. Every
blocking concern must be fixed, disproved with evidence, or routed as a genuine
operator decision. Re-review is conditional, not automatic.

Step 3 simplifies only after behavior is green, preserves any required
High-risk review or operator approval, and checkpoints the clean simplified
state before archival. Completion previews one explicit, hyphen-bounded
archive scope and applies that identical scope only after verifying the exact
milestone, implementation log, and test matrix. Relationships provide context,
not archive permission. The tracker then moves to `complete`, status is updated,
and `sync-and-commit` verifies the evidence without editing archived artifacts.
The closeout contract includes narrow recovery paths for an interruption after
the archive has moved files; it never fabricates missing command output or
replays an already-applied archive.

Whenever an operator decision is genuinely necessary, suite-maintained skills
explain where the question came from, why it matters now, practical effects, a
grounded example, and a recommendation when evidence supports one. This changes
how questions are presented, not which choices belong to the operator.

## What adoption costs

Teams must accept the docs convention, keep project context useful, work in
milestone-sized slices, and treat documentation validation as part of delivery.
Autonomous runs also spend up to two initial independent review calls after
each of Steps 0–2, and using both provider families requires separately
authorized access.

The suite fits substantial features, greenfield products, internal tools, and
projects that pause, resume, or change hands. It is usually too much process for
tiny fixes and one-off scripts.

The result is not unattended development. The operator still owns product
tradeoffs, High-risk approvals, genuine ambiguity, and final branch review. The
suite gives those decisions durable context—and gives the next agent a reliable
place to begin.
