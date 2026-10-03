# How I run Claude Code on multi-week projects

Lifecycle: active
Role: notes
Project: agent-playbook-suite
Updated: 2026-10-03

One Claude Code session decides the API. The next writes tests against a
different shape. A week later, a fresh agent reads the inconsistent repository
and asks you a question you already answered. The hard part of a long agent-led
project is not code generation. It is continuity.

[Agent Playbook Suite](https://github.com/ArtRichards/agent-playbook-suite) is
one plugin for Claude Code and Codex that gives the agent a repeatable delivery
process. [`docs-cli`](https://github.com/ArtRichards/docs-cli) is the small
Python CLI underneath it. Together they keep decisions, plans, progress, and
open questions on disk instead of leaving project state in chat.

This post shows how the workflow fits together, what it costs, and where it
earns that cost.

## Try it

```sh
python3 -m pip install --upgrade docs-cli
claude plugin marketplace add ArtRichards/agent-playbook-suite
claude plugin install agent-playbook-suite@agent-playbook-suite
```

Prefer an isolated environment? Use `pipx install docs-cli`, or
`pipx upgrade docs-cli` for an existing pipx install. Make sure `docs` is on
`PATH` for the agent session.

(Codex: `codex plugin marketplace add ArtRichards/agent-playbook-suite --ref main`, then `codex plugin add agent-playbook-suite@agent-playbook-suite`.)

Restart the agent after installing the plugin so its skill index reloads.

New project: run [`project-foundation`](https://github.com/ArtRichards/agent-playbook-suite/tree/main/plugins/agent-playbook-suite/skills/project-foundation)
and drive the first milestone with
[`create-milestones`](https://github.com/ArtRichards/agent-playbook-suite/tree/main/plugins/agent-playbook-suite/skills/create-milestones).
Existing repo: start with `docs migrate` on your Markdown folder and judge the
convention on its own before adopting the workflow. The rest of this post
explains what you just installed.

## Using the playbook suite

The suite provides eight workflow skills plus `docs`. `use-cases` is optional,
`create-milestones` and `ship-milestone` are alternative delivery drivers, and
[`explore`](https://github.com/ArtRichards/agent-playbook-suite/tree/main/plugins/agent-playbook-suite/skills/explore)
is an optional cross-cutting capability for the cases where the technical route
is genuinely uncertain.

**1. Start the project once with `project-foundation`.** The agent inspects the repo, then proposes drafts for the charter, scope, architecture, milestone plan, and test strategy. Risk defaults to Standard; the agent asks before classifying work as High. The output is a docs tree plus new agent context, an additions file when existing instructions are protected, or confirmation that the context is complete. Five living docs track status, foundation progress, risks, engineering follow-ups, and feedback.

**2. Pin the use cases with [`use-cases`](https://github.com/ArtRichards/agent-playbook-suite/tree/main/plugins/agent-playbook-suite/skills/use-cases).** It runs automatically when foundation finishes. You and the agent work out how a user would actually use the product, and the primary cases land in `use-cases.md`. Tests focus on these first. It is optional, but skipping it removes their main anchor.

**3. Keep identity separate from order with [`manage-milestone-tracker`](https://github.com/ArtRichards/agent-playbook-suite/tree/main/plugins/agent-playbook-suite/skills/manage-milestone-tracker).** Use it for direct tracker operations; foundation and both delivery drivers enforce the same contract. Milestones use stable slugs such as `fetch-and-parse`; `milestone-plan.md` stores order, state, and dependencies. Reordering changes an integer; inserting adds a row without renaming existing files or branches. Both delivery skills derive `next` from the same eligible rows.

**4. Run each milestone with `create-milestones` or [`ship-milestone`](https://github.com/ArtRichards/agent-playbook-suite/tree/main/plugins/agent-playbook-suite/skills/ship-milestone).** Both use ten phases: define the contract, write RED tests, create fixtures, capture the RED baseline, update base interfaces, implement the core path, update wrappers, run GREEN, integrate and dogfood, then finish quality, docs, and refactoring. Pure refactors may use adequate existing GREEN tests as the baseline, with the same review and approval checkpoints. `create-milestones` asks you to confirm each phase. `ship-milestone` runs up to four autonomous steps: Step 0 creates missing artifacts when needed; Step 1 runs phases 1–4; Step 2 runs phases 5–10; Step 3 simplifies and closes. Its conductor runs on Claude Fable 5.1 (Claude Code's `fable` alias) and delegates to fresh sub-agents on stacked branches: Claude Opus 5.5 through the `opus` alias, or GPT-6 Astra (`gpt-6-astra`) in Codex, both at `xhigh` reasoning. If a host substitutes a model, the run records the substitution instead of claiming the requested model ran. Steps 0–2 attempt isolated Claude- and GPT-family reviews, and nothing merges to `main` without you.

**5. Wrap every step with [`sync-and-commit`](https://github.com/ArtRichards/agent-playbook-suite/tree/main/plugins/agent-playbook-suite/skills/sync-and-commit).** It verifies the work against the milestone and tracker, syncs the docs tree to reality, commits, and pushes when the current feature branch, remote, and project policy allow it. It does not bypass git hooks on its own and never pushes `main`. If the docs disagree with the code, its commit does not happen.

**6. Close each milestone with [`simplify`](https://github.com/ArtRichards/agent-playbook-suite/tree/main/plugins/agent-playbook-suite/skills/simplify).** This is a behavior-preserving cleanup pass; if nothing simplifies, it changes nothing. High-risk code changes get fresh review when available, with operator approval as fallback. Autonomous shipping records a clean pre-archive checkpoint, committing staged changes when present; previews the exact scope; archives only completed-milestone artifacts; marks the tracker row complete; and runs `docs check` on the resulting tree. The checkpoint makes interrupted closeout recoverable.

**Cross-cutting: use `explore` only for clear technical uncertainty.** It is not
a mandatory stage and does not trigger because code is unfamiliar. After
repository inspection, it compares plausible mechanisms when an
acceptance-critical assumption remains unverified, no pattern fits, or evidence
invalidates the selected route. It may run disposable probes, but does not write
production code or persist dependency or configuration changes. Weak evidence
produces an explicit gap or operator decision, not a guess.

Two small conventions do a lot of the lifting. Anything that surfaces mid-flight — feedback, an idea, a deferred check — goes into the project logs, not into the tail of a milestone doc that is about to be archived. And milestone meaning lives in a semantic slug while order lives in the tracker, so inserting or rearranging work never renames activated files, branches, or history.

When a workflow needs your decision, it explains why it matters, what each
option changes, and, when evidence supports one, its recommendation; otherwise
it says there is no strong recommendation. Routine, reversible choices do not
interrupt you.

## Using docs-cli

The skills manage documents through `docs`, a stdlib-only Python CLI (Python 3.11+, on PyPI). Managed documents require `Lifecycle`, `Role`, and `Updated`; `Project` ownership and typed `Related` edges are optional. The root `INDEX.md` is generated, not managed, and never hand-edited. In an initialized project docs root, the verbs are mechanical:

```sh
docs new milestone fetch-and-parse --project demo --title "Fetch and parse"
docs touch fetch-and-parse.md --check
docs archive fetch-and-parse.md --reason "Milestone complete"
```

In a shared docs root, the suite qualifies project directories, artifact stems,
archive scopes, and branch prefixes. Two projects can therefore use the same
milestone slug without colliding.

`docs check` validates the whole tree: missing fields, broken links, and lifecycle drift. With an explicit `--stale` window or a configured default, it also flags stale active docs. Interactive milestone work runs that gate at phase boundaries; autonomous shipping runs it at step boundaries.

You can use docs-cli without the suite. `docs install-skill` puts the bundled skill on any Claude Code host, and `docs migrate ./notes/` adopts an existing Markdown folder into the convention — dry-run by default, so you read the plan before anything changes. Many repos benefit from just the managed tree and the check gate; the milestone workflow is an optional layer on top.

Why a CLI instead of just telling the agent to keep Markdown tidy? Because consistency is exactly what agents are bad at across sessions. The CLI makes structural invariants mechanical and drift visible, so agents spend fewer tokens re-deriving the project's shape.

## Why the structure earns its keep

The workflow records decisions, plans, phase results, and open questions as they happen. Resuming starts from the canonical tracker and durable artifacts, not transcript archaeology.

Three more reasons the structure earns its keep:

- **Independent, cross-provider review.** At the end of Steps 0–2, `ship-milestone` freezes one evidence packet and independently attempts one fresh Claude-family and one fresh GPT-family reviewer, Opus 5.5 and GPT-6 Astra by default. Neither sees the other's conclusions first. If one provider is unavailable, it uses one reviewer from the available provider, not a same-provider substitute. Blockers must be fixed, disproven with evidence, or routed to you. Review repeats only when evidence cannot close a concern, a correction materially changes the approach, or concerns conflict.
- **Tests tied to behavior.** Tests should catch wrong behavior and survive safe refactors. Freeze exact bytes only when those bytes are part of the contract. Risk and concrete gaps determine additional validation.
- **No dead artifacts.** Every doc, log, and phase output is supposed to have a live downstream consumer — a demand-driven chain the suite calls information liveness. A milestone contract feeds the test matrix; the status file feeds the next session; a decision note feeds the reviewer. An artifact nobody reads is not neutral; it is a smell: either it is missing a consumer or it should not exist. This is the same discipline that makes a week-later handoff cheap, applied to the docs themselves.

## Make each check earn its cost

Dan Luu's [agentic-testing experiments](https://danluu.com/agentic-testing)
show how elaborate testing plans can yield weak evidence. One example used
palindromic inputs to test bitstream reversal. For a simple illustration, `aba`
is still `aba` when a broken implementation returns its input. Use `abc` and
expect `cba`, and the same test can expose the mistake. His results concern how
agents applied the techniques;
they do not establish that TDD, property testing, or formal methods are
inherently ineffective.

The suite's rule is: **Before counting a check as evidence, establish what wrong
answer it can reject.** Ordinary inspection or the existing RED run usually
answers that. Replacing an uninformative fixture can be enough; no new mutation
check or per-test record is required.

Expected answers come from agreed behavior or a trustworthy reference. An agent
copying its implementation's output into a fixture can preserve the same error
on both sides. When a test fails, establish whether the behavior, expectation,
or setup is wrong before changing code or assertions.

Foundation work identifies observable acceptance criteria and reuses the
project's test commands. Milestones reuse adequate tests; simplification and
sync can reuse results while their relevant inputs remain unchanged, unless
project policy requires a fresh run. Once selected checks pass and concrete
concerns are resolved, stop expanding testing. Add deeper checks for a
demonstrated gap or an existing project requirement.

## What it costs

- You learn a small, opinionated convention: one metadata block per managed file, a typed link graph, a generated index.
- You work in milestone-sized slices. Drive-by edits get little value from the full workflow.
- The docs tree is part of the build. `sync-and-commit` blocks commits that diverge from it.
- Genuinely novel work can spend extra agent and probe time in `explore`; ordinary work stays on the direct delivery path.
- `ship-milestone` can add two initial review calls after each of Steps 0–2, plus conditional re-review. The plugin supplies no accounts or credentials; dual review requires both provider mechanisms to be authorized.

For a one-file fix, this is over-engineered — keep the rationale in chat and move on. It pays off when restart cost is high: greenfield builds, multi-week features, any project a different agent will resume later.

## Three examples

### A new tool, from nothing

Say you want a CLI that crawls a site and reports broken links. In an empty repo, you ask the agent to start a project; `project-foundation` picks up, looks at what exists (nothing yet), and asks the front-half questions one phase at a time: what problem, for whom, what is out of scope, what are the milestones, how will you know it works. An abridged view of the resulting tree:

```
docs/specs/
├── charter.md            what we're building and why
├── scope-and-constraints.md
├── architecture.md
├── milestone-plan.md     semantic slugs, explicit order/state/dependencies
├── test-strategy.md      risk levels and gates
├── definition-of-ready.md
├── use-cases.md          how a user will actually use it
├── followup-log.md       open engineering items
├── feedback-log.md       feedback and ideas, as they come up
├── status.md             current milestone and phase
└── INDEX.md              generated map of all of it
```

Then you say "start fetch-and-parse." The agent claims that tracker row, creates a milestone doc (the contract: inputs, outputs, error cases, what done means), a test matrix mapped to your use cases, and an implementation log — and walks the ten phases, recording each one as it lands.

This is not hypothetical: docs-cli was built this way, and its repo keeps the full trail of plans and logs in [its `docs/` directory](https://github.com/ArtRichards/docs-cli/tree/main/docs). My favorite artifact in there: mid-project, milestone M6 was reframed from "publish to PyPI" to "preparation only," and the reframe is a one-paragraph note written the moment it was decided. A normal code diff would not tell you that — commits record what shipped more reliably than what you deliberately decided not to ship.

### An existing repo with a messy notes folder

You don't need a new project to start. If you have a `notes/` or `docs/` folder of accumulated Markdown, adopt it:

```sh
echo 'fixtures/' >> ./notes/.docsignore
docs migrate ./notes/             # dry run: per-file plan with inferred
                                  # role, lifecycle, and confidence
docs migrate ./notes/ --apply     # stamp metadata and write .docs.toml
docs index --root ./notes/        # generate INDEX.md
docs check ./notes/               # validate the result
```

The dry run shows the plan and confidence for every file, so you triage ambiguity before the migration changes Markdown. After `--apply` and `docs index`, each adopted Markdown file has inferred metadata, the folder has a generated index, and `docs check` is ready to add as a CI gate. Install the standalone skill with `docs install-skill` if you want an agent to maintain the tree — no suite or milestones required.

### Coming back after a week

The payoff case. You shelved a project mid-milestone; today you (or a different agent, or a different machine) pick it up. The session starts with a compact orientation set:

```
CLAUDE.md/AGENTS.md → project commands, rules, and branch conventions
milestone-plan.md  → session-storage is active at order 200
status.md          → "Session storage. Phase 6 of 10. Last completed:
                      Phase 5, base interfaces, 2026-06-05."
session-storage.md → the contract, deliverables, and open questions
session-storage-impl.md → every phase entry and decision so far
session-storage-test-matrix.md → contract-to-test coverage and gates
```

Those files are the durable orientation layer, not the whole evidence set. The agent also checks the relevant specs and use cases, current code and tests, branch state, and any review ledger before resuming. No transcript archaeology, no "let me re-explain the architecture." The sync boundary keeps the durable record current before it commits.

The test that matters: can a fresh session pick up your project without you explaining it? That is what this is for.

## Try it on a real project

Install the current CLI from PyPI with pip below, or use
`pipx install docs-cli` for an isolated install (`pipx upgrade docs-cli` to
update it). Then install Agent Playbook Suite from this repository's Codex or
Claude Code marketplace:

```sh
python3 -m pip install --upgrade docs-cli

# Codex
codex plugin marketplace add ArtRichards/agent-playbook-suite --ref main
codex plugin add agent-playbook-suite@agent-playbook-suite

# Claude Code
claude plugin marketplace add ArtRichards/agent-playbook-suite
claude plugin install agent-playbook-suite@agent-playbook-suite
```

Restart the agent so the newly installed skill index is loaded.

Pick one real, bounded milestone and see whether the next fresh session can
resume it from the repository without an explanation.
