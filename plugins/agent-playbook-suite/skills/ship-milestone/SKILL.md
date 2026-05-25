---
name: ship-milestone
description: Autonomously run a milestone through all ten TDD phases to completion. A lightweight conductor spawns fresh Opus sub-agents — milestone creation (if needed), then per-step planning, implementation, and fresh-eyes review — commits each step to its own branch, runs the same-instance consistency audit and /simplify. Use when the operator wants a milestone built end-to-end. Invoke as `/ship-milestone <milestone-id>` or `/ship-milestone next milestone`.
---

# ship-milestone

Drive one milestone through the full 10-phase TDD lifecycle —
contract, tests, fixtures, RED baseline, implementation, GREEN,
integration, quality, and a post-implementation simplify pass —
and finish with the work committed on a reviewable branch stack.

Assumes a project set up with the docs-cli + skills ecosystem
(see `project-foundation` / `create-milestones`). If the target
milestone's task plan and log do not exist yet, **Step 0** creates
them following the `create-milestones` conventions before the TDD
steps begin.

## Requirements

Required tooling:

- [`docs-cli`](https://github.com/ArtRichards/docs-cli) **v1.2.0
  (M7+)** for the `Lifecycle:` controlled-vocab field and expanded
  role vocab; **v1.3.0 (M8+)** for `docs new --body-from -|<path>`.

Companion skills (sub-agents invoke them by name):

- `create-milestones` — read by Step 0's milestone-creation agent
  for conventions when scaffolding a missing task plan.
- `docs` (the docs-cli skill) — used by every sub-agent for doc
  lifecycle and validation.
- `sync-and-commit` — called by each implementation agent at the
  end of its step (after fresh-eyes review feedback is applied).
- `simplify` — called by the Step 3 simplify agent.
- `code-review` (built-in) — optional, available to fresh-eyes
  review agents.

If a companion skill is missing, the sub-agent must fall back to
the manual equivalent and note it in the milestone log.

## Substance lives in references/

This SKILL.md is intentionally short. The substance lives in:

- [`references/agent-prompts.md`](references/agent-prompts.md) —
  the five sub-agent prompt templates (milestone-creation,
  planning, implementation, fresh-eyes review, simplify) with
  their resume-message variants.
- [`references/consistency-check.md`](references/consistency-check.md)
  — the same-instance audit each implementation agent runs after
  the last phase of its step.

## When this applies

The operator wants a milestone built end-to-end without
step-by-step interaction — they invoke
`/ship-milestone <milestone-id>` or `/ship-milestone next
milestone` and the conductor runs the full lifecycle.

Do **not** apply when:

- The user wants interactive milestone work — use
  `create-milestones` instead.
- The project has no docs tree yet — redirect to
  `project-foundation`.
- The user wants only one phase or one step — drive it manually
  rather than spinning up the full conductor.

## The conductor model

The session running this skill is the **conductor**. The
conductor does not implement, audit, simplify, or edit project
code or docs. It only:

- resolves and tracks the milestone (reading `status.md` / the
  milestone doc and log is allowed — that's coordination, not
  implementation);
- creates and checks out branches;
- spawns fresh sub-agents and sequences them;
- triages questions and findings, and runs `AskUserQuestion`;
- runs read-only end-of-run verification.

Every heavyweight unit of work — creating the milestone,
planning, implementing, reviewing, simplifying — is a **fresh
Opus sub-agent**. This keeps the conductor's context small and
bounded across the entire run, and means each step's agents are
automatically free of any prior step's context: they rebuild
understanding from the specs, logs, and code on the branch.

Run the conductor session on the largest available model with
high thinking — its triage decisions need it.

## The steps

| Step | Phases | Branch |
|---|---|---|
| 0 — Create the milestone *(only if its task plan is missing)* | — | `<slug>/milestone-setup` (off the start commit) |
| 1 — Contract & RED baseline | 1–4 | `<slug>/phases-1-4` (off Step 0's branch, else the start commit) |
| 2 — Implement & ship | 5–10 | `<slug>/phases-5-10` (off step 1's branch) |
| 3 — Simplify | post-10 | `<slug>/simplify` (off step 2's branch) |

`<slug>` is a branch-safe milestone id (e.g. `m4`). The branches
stack; nothing is merged to `main` — the operator reviews and
merges the stack.

Step 0, when needed, runs one **milestone-creation** agent.
Steps 1 and 2 each run three fresh sub-agents in sequence:
**planning → implementation → fresh-eyes review**. Step 3 runs
one **simplify** agent.

## Procedure

### Resolve the milestone and pre-flight

1. **Resolve the milestone — never guess.**
   - An explicit id argument → use it.
   - The literal phrase `next milestone` → read `status.md`,
     take the first row whose lifecycle is `draft` (or marked
     pending in the milestone table).
   - No argument → `AskUserQuestion` for which milestone. Do
     not proceed without an explicit answer.
2. **Check whether it was created.** Look for the milestone's
   task-plan doc and log (e.g. `m4-*.md` and `m4-*-impl.md`).
   If they exist, Step 0 is skipped. If they don't, **Step 0**
   will create them.
3. **Working tree must be clean** (`git status`). If dirty,
   **stop** — do not stash, do not commit. Ask the operator
   to clean up.
4. Note whether a git remote exists, and the current `HEAD`
   (the start commit).

### Resume detection

A run can be interrupted. Before starting:

- Step 0 is complete iff the milestone's task plan and log
  already exist.
- Check which step branches exist (`<slug>/milestone-setup`,
  `<slug>/phases-1-4`, `<slug>/phases-5-10`, `<slug>/simplify`).
- Read the milestone log's phase table for which phases are
  logged complete.
- Start at the first step that is not fully complete. If all
  are complete, report that and stop.
- If a step is partially complete, pass its agents the phase
  range plus `phases already logged complete: {list} — verify
  and continue from the first incomplete phase`.

### Step 0 — Create the milestone (only if missing)

Run this **only** when the milestone's task plan and log do not
exist. Otherwise skip to Step 1.

1. Create + check out `<slug>/milestone-setup` off the start
   commit (or check out the existing branch if resuming).
2. **Spawn the milestone-creation agent** (Agent tool:
   `subagent_type: general-purpose`, `model: opus`) with the
   [Milestone-creation agent prompt](references/agent-prompts.md#milestone-creation-agent).
   Keep its agent id/name.
3. It returns a draft milestone task plan + log and an `OPEN
   QUESTIONS` list. **Triage** the questions (see *Triage
   rules*); `AskUserQuestion` for genuine scope or contract
   forks.
4. **Resume the agent** (SendMessage) with the operator's
   answers: it finalizes the milestone doc + log, updates
   `status.md`, regenerates the docs INDEX in lockstep,
   confirms `docs check` is clean, and runs the
   `sync-and-commit` skill.
5. Step 1 now branches off `<slug>/milestone-setup` instead of
   the start commit.

Step 0 gets no separate fresh-eyes review — Step 1's planning
agent pressure-tests the milestone spec as part of its job.

### Step 1 — Contract & RED baseline (phases 1–4)

1. Create + check out `<slug>/phases-1-4` off
   `<slug>/milestone-setup` if Step 0 ran, otherwise off the
   start commit (or check out the existing branch if
   resuming).
2. **Spawn the planning agent** (`subagent_type: Plan`,
   `model: opus`) with the
   [Planning agent prompt](references/agent-prompts.md#planning-agent),
   `phase_range` = phases 1–4.
3. The planning agent returns a plan and an `OPEN QUESTIONS`
   list. **Triage** each question (see *Triage rules*):
   auto-resolve doc/spec/conventional ones and record the
   decision; for genuine requirement or scope forks, call
   `AskUserQuestion`.
4. **Spawn the implementation agent** (`subagent_type:
   general-purpose`, `model: opus`) with the
   [Implementation agent prompt](references/agent-prompts.md#implementation-agent),
   the finalized plan, and the resolved answers. Keep its
   agent id/name. It implements phases 1–4, commits per phase,
   runs the
   [same-instance consistency audit](references/consistency-check.md),
   and returns a summary **without** running sync-and-commit.
5. **Spawn the fresh-eyes review agent** (`subagent_type:
   general-purpose`, `model: opus`) with the
   [Fresh-eyes review agent prompt](references/agent-prompts.md#fresh-eyes-review-agent),
   reviewing `<slug>/phases-1-4` against its base. For Step 1
   it must specifically judge whether the phase-2 tests
   genuinely pin the contract.
6. **Triage** the review findings. Then **resume the
   implementation agent** (SendMessage) with the review
   findings to address (blockers + should-fixes), any operator
   answers to items it surfaced, and the instruction to then
   run the `sync-and-commit` skill. If the review was clean,
   the SendMessage simply says so and instructs
   sync-and-commit.
7. The implementation agent is on its own milestone branch, so
   `sync-and-commit` may push if a remote exists.

### Step 2 — Implement & ship (phases 5–10)

1. Create + check out `<slug>/phases-5-10` off
   `<slug>/phases-1-4`.
2. Run the **same three-agent sequence** as Step 1, with
   `phase_range` = phases 5–10. The planning agent is freshly
   spawned — it has none of Step 1's context and rebuilds it
   from the milestone doc, the now-updated log (phases 1–4 are
   logged), the specs, and the code on the branch.
3. The fresh-eyes review for Step 2 judges correctness,
   completeness against the milestone's Deliverables/Success
   Criteria, and that the suite is GREEN.
4. Same triage → resume → sync-and-commit as Step 1.

### Step 3 — Simplify (post-phase-10)

1. Create + check out `<slug>/simplify` off
   `<slug>/phases-5-10`.
2. **Spawn the simplify agent** (`subagent_type:
   general-purpose`, `model: opus`) with the
   [Simplify agent prompt](references/agent-prompts.md#simplify-agent).
   No planning agent, no review agent — `/simplify` is
   behavior-preserving and self-tests.
3. The simplify agent runs the `/simplify` process, confirms
   the suite is GREEN, and runs `sync-and-commit`. If nothing
   genuinely simplifies, it makes no changes —
   `sync-and-commit` then finds a clean tree, which is fine.

### End-of-run verification & report

The conductor now verifies directly (read-only — cheap, keeps
the final report first-hand rather than pure trust):

- run the test suite — expect fully GREEN;
- run the quality gate (lint, format check, type check);
- run `docs check` on the docs tree — expect exit 0;
- `git log --oneline` across the branch stack.

Then report to the operator: the branch stack created, what
shipped, every decision auto-resolved, every `AskUserQuestion`
asked and its answer, push status, and the verification
results. Note that the branches are left for the operator to
review and merge — nothing was merged to `main`.

## Triage rules

When a milestone-creation or planning agent surfaces an OPEN
QUESTION, an implementation agent surfaces a scope item, or a
review agent flags a finding:

- **Auto-resolve, do not ask** — documentation/spec
  inconsistencies, stale spec text, generated-artifact
  lockstep, naming, conventional choices with an obvious
  default, and clear bugs. Record the decision (in the
  milestone doc's Decisions or the log) and fold the fix into
  the relevant agent's work.
- **Ask the operator (`AskUserQuestion`)** — anything that
  changes milestone scope, alters intended behavior, or is a
  genuine requirement fork with no obvious right answer.
  Concentrate these in Step 0 and the planning stage so
  implementation runs unattended afterward.

## Stop conditions

Stop and surface to the operator — never loop, never relax a
test — when:

- the milestone cannot be resolved;
- the working tree is dirty at pre-flight;
- an implementation agent cannot reach the required test state
  (e.g. GREEN at phase 8);
- a review finding needs an operator decision (use
  `AskUserQuestion`);
- an agent reports a blocker it cannot resolve.

## Notes

- Sub-agents are always spawned `model: opus` with an explicit
  deep-thinking directive in the prompt (built into each
  template).
- The conductor owns all `AskUserQuestion` interaction;
  sub-agents surface questions by returning them, then are
  resumed (SendMessage) with the answers.
- Steps run straight through (0 → 1 → 2 → 3) — there is no
  pause at the RED baseline. The consistency audit and the
  fresh-eyes review are the quality gates;
  `AskUserQuestion` is the only thing that pauses the run.
- Push happens only via `sync-and-commit`, only when a remote
  exists, and only on the milestone branches — never on
  `main` or any shared branch.
