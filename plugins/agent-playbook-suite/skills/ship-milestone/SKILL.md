---
name: ship-milestone
description: Autonomously claim and run one semantic milestone through all ten TDD phases, isolated Claude-family and GPT-family reviews at the end of Steps 0-2, simplification, and explicit archive closeout. Uses milestone-plan.md for identity, order, state, dependencies, and deterministic next-work selection; commits each step to its own branch. Use when the operator wants a milestone built end-to-end. Invoke as `/ship-milestone session-storage` or `/ship-milestone next milestone`.
---

# ship-milestone

Drive one milestone through the full 10-phase TDD lifecycle — contract, tests,
fixtures, RED baseline, implementation, GREEN, integration, quality, and a
post-implementation simplify-and-close pass — then safely archive its artifact
set on a reviewable branch stack.

Assumes a project set up with docs-cli plus `project-foundation` /
`create-milestones`. Step 0 creates any missing task plan, implementation log,
or test matrix before TDD begins.

## Requirements

Required tooling:

- [`docs-cli`](https://github.com/ArtRichards/docs-cli) 2.0 or newer — for managed
  lifecycle, reciprocal relationships, link rebasing, archive preview/apply,
  and docs-tree validation.

Companion skills (sub-agents invoke them by name):

- `create-milestones` — read by Step 0's milestone-creation agent when
  scaffolding a missing task plan, implementation log, and test matrix.
- `explore` — invoked conditionally for a clearly uncertain or genuinely novel
  technical route, and for evidence-backed recovery from an invalidated route.
- `docs` (the docs-cli skill) — used by every sub-agent for doc lifecycle and
  validation.
- `sync-and-commit` — called by each implementation agent after step-review
  findings are applied.
- `simplify` — called by the Step 3 simplify-and-close writer.
- `code-review` (built-in) — optional for fresh-eyes review agents.

If a companion skill is missing, use the manual equivalent and note it in the
milestone log.

## Substance lives in references/

This SKILL.md is intentionally short. The substance lives in:

- [`references/agent-prompts.md`](references/agent-prompts.md) — prompt templates plus resume variants.
- [`references/consistency-check.md`](references/consistency-check.md) — the
  implementation agent's same-instance audit.
- [`references/review-protocol.md`](references/review-protocol.md) — the Step 0-2
  review and resume gate.
- [`../_shared/references/agentic-quality-model.md`](../_shared/references/agentic-quality-model.md)
  — the shared risk and solution-uncertainty model used by planning,
  implementation, review, and consistency checks.
- [`../_shared/references/operator-interaction.md`](../_shared/references/operator-interaction.md)
  — read before any operator question or decision.
- [`../_shared/references/milestone-tracker.md`](../_shared/references/milestone-tracker.md)
  — the normative semantic tracker contract; do not duplicate or weaken it.

The milestone-tracker contract is a hard requirement. Before pre-flight or
spawning a worker, verify and read it in full. If absent, stop, explain that the
full Agent Playbook Suite installation is incomplete, and recommend reinstalling
or updating it. Never reconstruct the contract from prompts, project artifacts,
or another skill, or use the companion-skill fallback for this reference.

Substitute every prompt `{placeholder}` before spawning: fresh agents cannot
resolve bundle-relative paths or unfilled tokens. `{skill_dir}` is this skill's
absolute directory (shown as "Base directory for this skill"); the bundled quality
model is `{skill_dir}/../_shared/references/agentic-quality-model.md`. If that
model is absent, tell the agent to use the milestone's QUALITY PLAN and test
matrix instead.

## When this applies

The operator wants an end-to-end run without step-by-step interaction: invoke
`/ship-milestone <milestone-slug>` or `/ship-milestone next milestone`.

Do **not** apply when:

- The user wants interactive milestone work — use `create-milestones` instead.
- The project has no docs tree yet — redirect to `project-foundation`.
- The user wants only one phase or step — drive it manually instead.

## The conductor model

The session running this skill is the **conductor**. It does not implement,
audit, simplify, or edit project code or docs. It only:

- resolves the canonical tracker row and inspects milestone artifacts;
- creates and checks out branches;
- spawns fresh sub-agents and sequences them;
- conditionally invokes `explore` and routes its disposition;
- triages questions and findings, and asks the operator
  (`AskUserQuestion` where available);
- runs read-only end-of-run verification.

Every heavyweight unit is a **fresh high-capability sub-agent**. For creation,
planning, implementation, and simplification, prefer Codex `gpt-6-astra`
`xhigh` or Claude Code's newest `opus` alias at `xhigh`. Record any effective
substitution. Fresh agents rebuild from branch artifacts; reviewers follow the
linked protocol and never fill a missing provider slot with a same-provider
substitute.

Run the conductor on Claude Code's `fable` alias (currently Claude Fable 5.1)
or Codex `gpt-6-astra`, with high reasoning for triage.

## The steps

| Step | Phases | Branch |
|---|---|---|
| 0 — Create the milestone *(only if its task plan is missing)* | — | `<project-branch-prefix><slug>/milestone-setup` (off the start commit) |
| 1 — Contract & RED baseline | 1–4 | `<project-branch-prefix><slug>/phases-1-4` (off Step 0's branch, else the start commit) |
| 2 — Implement & ship | 5–10 | `<project-branch-prefix><slug>/phases-5-10` (off step 1's branch) |
| 3 — Simplify & close | post-10 | `<project-branch-prefix><slug>/simplify` (off step 2's branch) |

`<slug>` is the project-unique semantic identity; its first `active` transition
freezes it, and activated numeric slugs stay valid. Project slugs are repository-globally
unique; use prefix `<project>/` with multiple suite projects, otherwise empty.
Reordering never renames the branch stack. Nothing is merged to `main` — the
operator reviews and merges the stack.

Before the first branch, a retained **claim writer** serializes selection and
changes the chosen tracker row from `planned` to `active`. Step 0, when needed,
runs one **milestone-creation** agent followed by its end-of-step reviews.
Steps 1 and 2 run **planning → implementation → isolated Claude/GPT review
attempts**. Step 3 runs one **simplify-and-close** writer. When the stable
contract and RED baseline exist and the
shared quality model's high-threshold solution-uncertainty gates are met, an
`explore` handoff runs before Step 2 implementation; it is not a routine fourth
agent or mandatory step.

## Procedure

### Resolve, claim, and pre-flight

1. Resolve `{docs_root}`, `<project-path>`, and `<artifact-stem>`; read the shared tracker contract and validate the canonical tracker. `status.md` is narrative context, never a scheduler.
2. Resolve exactly one row: an explicit semantic slug selects that row; `next
   milestone` selects the lexicographically first eligible `planned` slug at
   the lowest eligible `Order`. With no argument, ask under the shared
   operator-interaction policy. Never derive next work from filenames, row
   position, `status.md`, or ordinal-id sorting.
3. Inspect the row, readiness, dependencies, blockers, artifacts, collisions, and
   `<project-branch-prefix><slug>/...` branches before rejecting terminal state. A named `active`
   row is a resume candidate. Stop on `paused`, `cancelled`, or ineligible `planned`.
   A `complete` row is terminal unless read-only inspection proves an established
   Step 3 closeout with exactly the canonical three artifacts archived and final sync incomplete; otherwise it is unrelated or closed. Never rewrite state.
4. Require a clean tree, then note `HEAD` and remote; never stash or absorb work.
   The only dirty-tree exception is an interrupted Step 3 closeout on its
   simplify branch, with the row `active` or qualifying as `complete` above.
   Read-only inspection must verify the recorded clean pre-archive simplify
   checkpoint and find no code/unrelated-doc change after it; the frozen archive
   move/link-rewrite set plus unfinished tracker, status, or INDEX must explain
   every path. Preserve it, reconstruct apply evidence from its diff and archive
   witnesses, and route recovery directly to Step 3 resume detection. Do not
   switch branches, rerun earlier work/simplify/archive, or accept unexplained state.
5. For a `planned` row, retain the
   [claim writer](references/agent-prompts.md#milestone-claim-writer): it
   re-reads the tracker, fails on change, moves only this row to `active`, runs
   `docs touch` / `docs check`, and returns the uncommitted checked diff.
6. Only after the row is observably `active`, create the first required branch
   while carrying that exact claim diff. Resume the same writer to commit the
   claim on that branch and return a clean tree before any production agent is
   launched. For an already-`active` resume, verify the existing claim and use
   its established branch stack; reconcile a missing or divergent claim
   checkpoint before proceeding.

### Resume detection

A run can be interrupted. Before starting:

- Require `active` for ordinary resume; permit `complete` only for verified exact-set
  Step 3 final-sync recovery. Match its slug to every path; never re-claim or rename.
- Step 0 is complete only if linked artifacts predated this ship run with no
  `<project-branch-prefix><slug>/milestone-setup` branch/ledger, or its in-flight ledger meets the
  review protocol including `final-sync: ready` in the clean branch `HEAD`.
  Artifacts alone never close an in-flight Step 0.
- Check which step branches exist under `<project-branch-prefix><slug>/` and read
  the milestone log's phase table for completed phases.
- Read recorded exploration dispositions; do not rediscover rejected/blocked routes.
- Apply review-ledger resume rules. Preserve repeated Step 1 work as append-only
  `review-pass-N -> contract-rework-N -> review-pass-(N+1)`, targeted re-review
  inside its pass, each prior packet, and the exact current impl-log path#anchor
  plus predecessor chain. Never combine frozen heads or repeat a successful attempt.
- If the implementation log contains an open `CONTRACT CHANGE REQUIRED`
  marker, run contract-change recovery before ordinary step/phase completion
  detection; completed phases 1-4 do not override that marker.
- Detect closeout first: active artifacts mean archive has not applied; exactly
  the canonical three archived artifacts with matching witnesses and an
  `active` row route to tracker/status completion; that set with a `complete`
  row and incomplete final sync routes to docs verification and final sync. A
  clean Step 3 `HEAD` proving archive, state completion, and final sync is done;
  paused, cancelled, and unrelated complete rows remain terminal. Never rerun archive.
- For any post-archive route, require the recorded simplify checkpoint commit/tree
  and its High-risk review/approval evidence when applicable; verify the commit
  is on the simplify branch and its diff to current state contains no code change.
- Start at the first step that is not fully complete. If all
  are complete, report that and stop.
- If a step is partially complete, pass its agents the phase
  range plus `phases already logged complete: {list} — verify
  and continue from the first incomplete phase`.

### Step 0 — Create the milestone (only if missing)

Run this when an artifact is missing/unlinked or an existing
`<project-branch-prefix><slug>/milestone-setup` branch/ledger is not closed. Skip only when linked
artifacts predated this run and no in-flight Step 0 state exists.

The Step 0 artifact set is the task plan, implementation log, and test matrix;
all three must be linked with `Related: pairs-with`.
On resume, start at the ledger's first incomplete state; do not repeat work.

1. Use the existing `<project-branch-prefix><slug>/milestone-setup` branch (or its
   existing resume branch).
2. **Spawn the milestone-creation agent** using the host's
   worker/general-purpose sub-agent on the strongest available
   model under [The conductor model](#the-conductor-model) policy with the
   [Milestone-creation agent prompt](references/agent-prompts.md#milestone-creation-agent).
   Keep its agent id/name.
3. It returns a draft milestone task plan, implementation log, test
   matrix, and an `OPEN QUESTIONS` list. **Triage** the questions
   (see *Triage rules*); ask the operator for genuine scope or
   contract forks.
4. **Resume the agent** (SendMessage) with the operator's
   answers: it finalizes the milestone doc, implementation log, test
   matrix, links the active tracker row, synchronizes its relationships, keeps
   `status.md` narrative-only, regenerates INDEX,
   confirms `docs check` is clean, initializes the Step 0 review ledger,
   and creates the clean review-ready checkpoint required by the protocol.
5. Run the [Step review protocol](references/review-protocol.md) against that
   frozen commit with the
   [fresh-eyes prompt](references/agent-prompts.md#fresh-eyes-review-agent) and
   Step 0 test-quality clause.
6. Follow the protocol's triage, retained-writer checkpoint, conditional
   re-review, and resume-ledger rules. If no review returns, record both
   unavailable outcomes and stop. Otherwise use the
   [finalization message](references/agent-prompts.md#step-review-finalization)
   to invoke `sync-and-commit` with explicit `ship-milestone Step 0` context.
7. Step 1 now branches off `<project-branch-prefix><slug>/milestone-setup` instead of
   the start commit.

### Conditional exploration handoff

The Step 1 planner stabilizes the contract and RED evidence; it does not trigger
technical route exploration. The Step 2 planning agent returns an `EXPLORATION
SIGNAL`. `NONE` continues the ordinary sequence. For a proposed signal, the
conductor checks the shared quality model's automatic-trigger gates; ordinary
unfamiliarity, low confidence, or a transient failed command does not qualify.

When the gates are met:

1. Spawn a fresh worker/general-purpose agent under
   [The conductor model](#the-conductor-model) policy and instruct it to invoke
   the `explore` skill with the planning agent's decision packet. Keep both
   agent ids until the handoff is resolved. Exploration owns no production
   implementation and keeps one writer for any durable record.
2. Keep the full registry and evidence in the milestone implementation log;
   put only the disposition and a link in the milestone's Decisions section.
   Exploration may identify a behavior, scope, or fixed-constraint change as
   the only unlock, but it does not edit the milestone contract or RED tests,
   and that finding is not approval to change them. Add an open `CONTRACT
   CHANGE REQUIRED` marker only after either a resumed planner returns it for a
   `SELECTED` route or the contract owner explicitly approves the exact named
   change after a `NO VIABLE ROUTE` gap.
   Before routing any disposition, have the exploration agent run the docs
   lifecycle/check commands and create an exploration checkpoint commit on the
   current milestone branch. Do not run `sync-and-commit` or push; return only
   after the working tree is clean.
3. Route the disposition:
   - `SELECTED` — resume the planning agent with the evidence-backed route and
     the [planning resume message](references/agent-prompts.md#planning-resume-after-exploration).
     If it returns `CONTRACT CHANGE REQUIRED`, resume the retained exploration
     agent with the
     [marker-checkpoint message](references/agent-prompts.md#contract-change-marker-checkpoint)
     before starting contract-change recovery.
   - `OPERATOR DECISION` — ask only for the product value or scope choice that
     evidence cannot settle, then use the
     [exploration resume message](references/agent-prompts.md#exploration-resume-after-an-operator-decision).
     Require it to update the canonical records, run the docs lifecycle/check
     commands, create a new exploration checkpoint commit, and return only with
     a clean tree. Route its returned final disposition through this list
     before taking any further action; only a returned `SELECTED` may resume
     planning.
   - `NO VIABLE ROUTE` — stop exploration and ordinary Step 2 planning. Surface
     the route registry and exact blocking clause or fixed constraint to the
     contract owner. If the owner preserves the contract or does not approve
     the named change, stop the milestone. Only after the contract owner
     explicitly approves changing the named clause or constraint, resume the
     retained exploration agent with the marker-checkpoint message to record
     the approval and open marker in a clean checkpoint, then enter
     contract-change recovery.
   - `INSUFFICIENT EVIDENCE` — stop and surface the registry, exact evidence
     gap, and unlock condition. Do not enter contract-change recovery merely
     because evidence is unavailable.
4. Exploration never edits the milestone contract, fixed constraints, or RED
   tests. It records evidence, dispositions, approvals, and the open marker;
   contract-change recovery owns any resulting contract and test edits.

If an implementation agent later returns a qualifying blocker dossier, require
its distinct WIP/blocker checkpoint commit and a clean tree before invoking
`explore` in `recovery` mode. The exploration checkpoint then contains only the
canonical exploration records. A selected alternate route goes through a fresh
planning pass that accounts for the WIP commit before a fresh implementation
agent resumes; never discard partial work or repeat the failed mechanism without
the new evidence required by its reopen rule.

### Contract-change recovery

An open `CONTRACT CHANGE REQUIRED` marker takes precedence over normal resume
detection. Stay on the current phases-5-10 branch so the exploration checkpoint
remains in its history, but do not begin production implementation:

1. Spawn a fresh planning agent for phases 1-4 with the exploration handoff and
   the exact contract gap. Triage any behavior or scope choice through the
   operator as usual.
2. Run the ordinary Step 1 implementation, isolated two-provider review
   protocol, triage, and risk-aware RED-baseline checkpoint on the corrected
   contract and tests. Log these as contract-rework entries rather than
   erasing the earlier phase history.
3. Close the marker only after the corrected contract is stable, the Phase 4
   baseline meets the TDD phase criteria, review findings are resolved, and
   any High-risk approval is recorded.
4. Spawn a fresh Step 2 planning agent and process its exploration signal from
   the corrected artifacts. If interrupted before the marker closes, resume
   this recovery sequence rather than skipping to implementation.

### Step 1 — Contract & RED baseline (phases 1–4)

The baseline is RED for new or corrected behavior; a pure refactor may use
adequate GREEN coverage under the TDD phase reference. The same review and
High-risk approval checkpoint applies to either baseline.

1. If Step 0 ran, create `<project-branch-prefix><slug>/phases-1-4` off
   `<project-branch-prefix><slug>/milestone-setup`. If Step 0 was unnecessary, use the
   phases-1-4 branch already created around the claim checkpoint. Check out an
   existing branch when resuming.
2. **Spawn the planning agent** using the host's planning-capable
   sub-agent under [The conductor model](#the-conductor-model) policy with the
   [Planning agent prompt](references/agent-prompts.md#planning-agent),
   `phase_range` = phases 1–4.
3. The planning agent returns a plan and an `OPEN QUESTIONS`
   list. **Triage** each question (see *Triage rules*): auto-resolve
   doc/spec/conventional ones and record the
   decision; for genuine requirement or scope forks, ask the operator.
4. **Spawn the implementation agent** using the host's worker/general-purpose sub-agent under
   [The conductor model](#the-conductor-model) policy with the
   [Implementation agent prompt](references/agent-prompts.md#implementation-agent),
   the finalized plan, and resolved answers. Keep its id/name. It implements
   phases 1–4, commits per phase, runs the
   [same-instance consistency audit](references/consistency-check.md),
   initializes the Step 1 review ledger, and returns a clean, committed
   review-ready `HEAD` **without** running `sync-and-commit`.
5. Freeze that `HEAD` and run the
   [Step review protocol](references/review-protocol.md) with the same
   [fresh-eyes prompt](references/agent-prompts.md#fresh-eyes-review-agent) and
   packet. For Step 1 the test-quality clause must specifically judge whether
   the phase-2 product tests or selected explicit non-product checks genuinely
   pin the contract.
6. Follow dual-result triage, retained-writer checkpoint, conditional re-review,
   and resume rules. If no review returns, record both unavailable and stop. Otherwise use the
   [finalization message](references/agent-prompts.md#step-review-finalization)
   to invoke `sync-and-commit` with explicit Step 1 context and the exact current
   qualified review-pass path#anchor; sync verifies its predecessor chain.
7. Apply the **risk-aware RED-baseline checkpoint** from the planning agent's
   `QUALITY PLAN`, the milestone doc, and all returned reviews' Risk-gate
   decisions:
   - **Lite:** continue automatically only if the tests genuinely pin the
     contract and all accepted blockers/should-fixes are resolved.
   - **Standard:** continue only if neither returned review has an unresolved
     blocker on contract/test adequacy and all accepted should-fixes are
     resolved.
   - **High:** stop after Step 1. Ask the operator to approve
     the contract, visible tests, hidden/generalization plan
     (categories only, no private cases), mock policy, and selected
     gates after review-closure sync and before Step 2. Resume only after
     explicit approval or completion of the requested Step 1 fixes.

### Step 2 — Implement & ship (phases 5–10)

1. Create + check out `<project-branch-prefix><slug>/phases-5-10` off
   `<project-branch-prefix><slug>/phases-1-4` only after the Step 1
   risk-aware RED-baseline checkpoint allows continuation.
2. Run the Step 1 planning and implementation sequence with
   `phase_range` = phases 5–10. The fresh planner rebuilds from updated
   artifacts/code; process its `EXPLORATION SIGNAL` before implementation.
3. Run the [Step review protocol](references/review-protocol.md) for Step 2.
   Every returned review judges correctness, completeness against the
   milestone's Deliverables/Success Criteria, and that the selected product
   tests plus configured explicit checks are GREEN.
4. Use the same dual-result triage, review-resolution checkpoint, conditional
   re-review, ledger finalization, and explicit `ship-milestone Step 2`
   `sync-and-commit` invocation as Step 1.

### Step 3 — Simplify & close (post-phase-10)

1. Create + check out `<project-branch-prefix><slug>/simplify` off `<project-branch-prefix><slug>/phases-5-10`.
2. **Spawn the simplify-and-close agent** using a fresh worker/general-purpose
   sub-agent under [The conductor model](#the-conductor-model) policy with the
   [Simplify agent prompt](references/agent-prompts.md#simplify-agent),
   filling `{unresolved_taste_findings}` with waived/deferred Step 1–2 taste
   findings (or "none"). There is no planner or routine Step 0–2 dual-provider review.
   If High-risk simplification changes code, satisfy `/simplify`'s single
   [fresh-reviewer](references/agent-prompts.md#high-risk-simplify-review-agent)-or-operator approval gate before the checkpoint.
3. Prove GREEN and finish active-doc updates; freeze the exact staged tree, then
   create and record a clean pre-archive simplify checkpoint whose tree matches it
   (or record clean current `HEAD` when nothing changed). Pass `{docs_root}` and
   change to it before docs-cli operations. Normal mode freezes paths/date/reason and literal
   `<project-path><artifact-stem>-*`, then JSON-previews/applies the exact set. In
   verified interruption recovery, never preview/archive; reconstruct the exact
   moved set and witnesses. If the row is `active`, change only State to
   `complete` and update/touch tracker/status; if already `complete`, preserve it.
   Resume the first incomplete `docs touch`/`docs index .`/`docs check .`/sync stage; relationships never authorize archive.
4. Invoke `sync-and-commit` with Step 3 context, evidence mode, checkpoint SHA/tree,
   gate evidence, and matching archive evidence; it never archives or touches archived docs.

### End-of-run verification & report

The conductor now verifies directly (read-only — cheap, keeps
the final report first-hand rather than pure trust):

- run the selected product test suite — expect GREEN;
- run configured explicit non-product checks — expect GREEN;
- run the configured quality gate (lint, format check, type check where present);
- run `docs check` on the docs tree — expect exit 0;
- verify the tracker row is `complete`, its rebased archive link resolves, and
  the exact primary/companion archive set has matching archive metadata;
- `git log --oneline` across the branch stack.

Report the branch stack, shipped behavior, decisions/questions and answers,
reviewer identities/unavailability, blocker dispositions, push status,
archive preview/apply evidence, and verification. Leave branches for operator
review and merge; never merge to `main`.

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
- **Ask the operator (`AskUserQuestion` where available)** — changes to
  milestone scope or intended behavior, or a genuine requirement fork with no
  clear answer. Apply the shared operator-interaction policy before asking;
  never forward a bare worker question. Concentrate these in Step 0 and
  planning.
- **Blockers from either review** — preserve their source ids and disposition
  each as `fixed`, `disproven` with contract/code/test evidence, or `operator
  decision` with the recorded answer and applied result. Reviewer agreement is
  not evidence and an unanswered operator decision remains open.
- **Taste findings (the review's `## Taste` section)** — must-triage,
  never dropped. Gating taste findings (anchor violations, scope
  creep, hygiene, library duplication — see the shared quality
  model's Taste model) are handled as blockers. For each
  subjective finding, the conductor decides **fix** or **waive
  with a recorded reason**: on Lite/Standard milestones the
  conductor waives on its own authority; on High-risk milestones
  every waiver goes to the operator. Pass
  all decisions to the implementation agent as
  `{taste_triage_decisions}` so the impl log's taste-triage table
  is complete before sync-and-commit.

## Stop conditions

Stop and surface to the operator — never loop, never relax a
test — when:

- the milestone cannot be resolved;
- the tracker is invalid, changed during claim, or disagrees with the selected
  semantic identity, state, dependencies, artifact set, or branch stack;
- the working tree is dirty at pre-flight, except for a forensically verified
  interrupted Step 3 closeout on its established simplify branch;
- an implementation agent cannot reach the selected test state and its blocker
  is not a qualifying solution-uncertainty signal, or `explore` recovery returns
  `NO VIABLE ROUTE` or `INSUFFICIENT EVIDENCE` that cannot be closed in the
  current run;
- a High-risk milestone has completed Step 1 and needs operator
  approval for the contract, visible tests, hidden/generalization
  plan, mock policy, and selected gates before Step 2;
- a review finding needs an operator decision (ask the operator);
- neither provider-family review returns for a Step 0, 1, or 2 packet, or any
  reviewer-labeled blocker lacks a completed disposition;
- a taste finding is left untriaged — no fixed/waived-with-reason
  decision recorded (a High-risk waiver additionally needs
  operator approval);
- exploration requires an operator-owned product decision (ask the operator)
  or all route families share the same external blocker;
- Step 3 lacks a clean pre-archive checkpoint, applicable High-risk approval, an
  exact archive preview/apply match, or recovery proof of no code change after
  the checkpoint; or verification would require editing an archived document.

## Notes

- Prompts carry the deep-reasoning directive; record substituted models. End-of-step reviewer selection follows the narrower provider protocol.
- The conductor owns operator questions; sub-agents return questions and resume
  with the answers. Use `AskUserQuestion` where available; otherwise ask in
  ordinary operator-visible conversation and wait for the answer.
- Lite/Standard continue only when the Step 1 RED checkpoint allows it. High
  pauses there for operator approval. The consistency audit and Step 0-2
  review protocol remain gates.
- Push happens only via `sync-and-commit`, only with a remote and on milestone
  branches — never on `main` or a shared branch.
