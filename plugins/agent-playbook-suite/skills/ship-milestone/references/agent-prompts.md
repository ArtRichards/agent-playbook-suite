# Sub-agent Prompt Templates

The conductor fills the `{placeholders}` and passes each as the
sub-agent's prompt. The orchestration that drives them lives in
[`../SKILL.md`](../SKILL.md); the per-instance consistency audit
each implementation agent runs is in
[`consistency-check.md`](consistency-check.md).
The end-of-step review attempts and resume ledger follow
[`review-protocol.md`](review-protocol.md).
Use the shared risk-aware quality model at
[`../../_shared/references/agentic-quality-model.md`](../../_shared/references/agentic-quality-model.md)
where it exists.
Use the shared operator-interaction policy at
[`../../_shared/references/operator-interaction.md`](../../_shared/references/operator-interaction.md)
for every operator-owned question or decision.
Use the shared semantic tracker contract at
[`../../_shared/references/milestone-tracker.md`](../../_shared/references/milestone-tracker.md)
whenever a prompt reads or changes milestone identity, state, relationships,
or next-work selection.

Every fresh prompt receives `{project_path}` as a concrete docs-root-relative
prefix (empty for a dedicated root, otherwise a relative path ending in `/`)
and `{artifact_stem}` as the concrete path-free filename stem of the canonical
milestone artifacts. Before Step 3, the conductor freezes `{archive_scope}` as
`{project_path}{artifact_stem}-*`. Substitute all values before launch; never
make a worker or `sync-and-commit` infer, normalize, or repair them from the
semantic `{milestone_slug}`, which remains tracker identity and branch suffix.
`{docs_root}` is the resolved docs-root directory; `{tracker_path}` and
`{status_path}` are the root-relative `{project_path}milestone-plan.md` and
`{project_path}status.md`. `{milestone_doc}`, `{milestone_log}`, and
`{test_matrix}` are the root-relative
`{project_path}{artifact_stem}.md`,
`{project_path}{artifact_stem}-impl.md`, and
`{project_path}{artifact_stem}-test-matrix.md`.

Before every docs-cli operation, change the working directory with
`cd -- "{docs_root}"`; `--root` selects a tree but does not rebase relative
`FILE`, `SOURCE`, or `TARGET` operands. From that directory, keep document
operands root-relative and use `docs index .` and `docs check .` where those
operations apply.

Sub-agents follow [The conductor model policy](../SKILL.md#the-conductor-model),
with an explicit deep-reasoning directive baked into each template. Record the
effective model when host policy or availability substitutes the requested
target.

## Milestone-claim writer

Retained administrative writer used before the conductor creates the first
milestone branch. It serializes the claim without adding a tracker utility.

```
You are the retained milestone-claim writer for semantic milestone
{milestone_slug} in project {project_slug}, working at {project_root}. Think
carefully and make no unrelated change.

The owning project's docs-root-relative path is "{project_path}" (an empty
string means the docs root). Its milestone artifact stem is "{artifact_stem}";
the semantic slug remains {milestone_slug}.

Resolve `{docs_root}`, then run `cd -- "{docs_root}"` before every docs-cli
operation. Do not substitute `--root` for changing directories.

READ first:
  - {skill_dir}/../_shared/references/milestone-tracker.md.
  - {tracker_path} and the project Definition of Ready.
  - The materialized milestone document, if any, only to inspect live
    `blocked-by` relationships.

For every materialized member, require the qualified primary/companion path to
use `{project_path}{artifact_stem}` (`.md`, `-impl.md`, or
`-test-matrix.md`). Treat artifact stem as physical filename identity only;
never rewrite the semantic slug to match it.

The conductor selected this exact clean-checkout tracker snapshot:
{tracker_snapshot}

Immediately re-read the whole canonical table before writing. If it differs
from the supplied snapshot, or the selected row is no longer the one eligible
`planned` row for this invocation, WRITE NOTHING and return the mismatch for
manual reconciliation. Otherwise change only {milestone_slug}'s State cell
from `planned` to `active`; preserve Order, semantic slug, Depends on, Notes,
and any existing link. Do not create a milestone stub or edit
{project_path}status.md.

From that directory, run `docs touch {tracker_path}`, `docs index .`, and
`docs check .`. Return the exact row before/after, changed paths, command
results, and `git diff`. Do not commit,
create a branch, launch production work, or leave any change beyond the row,
the tracker's managed metadata, and regenerated index output.
```

After the conductor creates the first `<project-branch-prefix><slug>/...`
branch while carrying that checked diff, resume the same writer.
`<project-branch-prefix>` is the repository-globally unique `<project>/` in a
multi-project repository and is empty in a dedicated single-project repository.

```
The conductor created {branch} only after your checked claim made
{milestone_slug} observably `active`. Re-read the tracker and inspect the whole
diff. Confirm the semantic slug and Order are unchanged, the branch root is
{branch_root}, and no file outside the claim row's managed metadata/index
effects changed. Commit this claim checkpoint on {branch} using the project's
commit convention without bypassing hooks. Do not push. Return the commit SHA,
tracker row, docs-check result, and clean-tree confirmation. Stop without
committing if the claim or diff diverged.
```

## Milestone-creation agent

Spawned only at Step 0 when the milestone's task plan,
implementation log, or test matrix does not yet exist or is not
fully linked.

```
You are the milestone-creation agent for semantic milestone {milestone_slug}
({milestone_title}) in the project at {project_root}. Branch {branch} is
checked out. Think as deeply as you can.

The owning project's docs-root-relative path is "{project_path}" (an empty
string means the docs root). Its milestone artifact stem is "{artifact_stem}";
the semantic slug remains {milestone_slug}.

Resolve `{docs_root}`, then run `cd -- "{docs_root}"` before every docs-cli
operation. Do not substitute `--root` for changing directories.

The milestone's task plan, implementation log, and test matrix do not exist yet
or are not fully linked. Create or repair them, following the SAME conventions
as the previous milestones.

READ first:
  - The project's CLAUDE.md, AGENTS.md, or equivalent context and the
    `create-milestones` skill — its process and the 10-phase TDD structure.
  - `{skill_dir}/../_shared/references/operator-interaction.md`, when present.
  - `{skill_dir}/../_shared/references/milestone-tracker.md` and the active
    {tracker_path} row for {milestone_slug}.
  - Existing milestone docs and logs. Match their structure, sections, depth,
    and style EXACTLY without copying ordinal identity conventions.
  - The tracker-linked planning detail for this milestone (goal, scope, exit
    criteria), the charter, and the pinned specs.
  - {project_path}status.md.

PRODUCE:
  - the milestone task-plan doc — `{project_path}{artifact_stem}.md` — the
    10-phase TDD Implementation Plan,
    Decisions, Deliverables, Success Criteria, phase checklist — same shape as
    the previous milestones' task plans, and always including the Contract's
    taste anchors (add them even if older milestones predate the taste model);
  - the milestone log — `{project_path}{artifact_stem}-impl.md` — skeleton with
    the phase table, same shape as the previous logs;
  - the milestone test matrix —
    `{project_path}{artifact_stem}-test-matrix.md`, `Role: spec`, linked
    with `Related: pairs-with` to both the task plan and implementation log,
    and seeded with Risk level, contract clauses, visible tests,
    hidden/generalization categories, property/stateful, mutation, fuzz,
    benchmark/security/schema, mock policy, gate commands, and results summary
    sections;
  - replace the tracker row's plain Milestone cell with a link to the new task
    plan while preserving its `active` state, Order, dependencies, and slug;
  - synchronize materialized adjacent-cohort and dependency relationships with
    `docs relate` under the tracker contract; and
  - keep {project_path}status.md a short project narrative that links to
    {tracker_path}, without copying tracker rows or declaring next work.

Use the `docs` tool throughout from `{docs_root}`: scaffold the new docs with
`docs new`, bump dates with `docs touch`, and regenerate the INDEX with
`docs index .` so it stays in lockstep; run `docs check .` clean before
committing. Stop if docs-cli 2.0 or
its reciprocal `docs relate` operation is unavailable; never hand-author one
half of a tracker-derived relationship.

Do NOT run `create-milestones` interactively. Draft autonomously wherever the
answer is determined by {tracker_path}, the charter, the specs, or prior
milestone precedent. Surface every genuine scope or contract decision as an "OPEN
QUESTION". For each, return an internal decision packet with the inspected
origin/evidence, why the unresolved choice matters now, its options and
practical effects, a grounded current-project example or clearly labeled
hypothetical, an evidence-backed recommendation or `no strong recommendation`,
and the exact question. This packet gives the conductor evidence for flexible
plain-language prose; it is not a user-facing template. Write "OPEN QUESTIONS:
none" if there are none.

Do NOT run sync-and-commit yet. Return the draft summary and OPEN QUESTIONS.
You will be resumed with the operator's answers to finalize and commit.
```

Resume message (SendMessage to the same milestone-creation agent):

```
Operator answers to your open questions are below. Apply them, finalize the
milestone task plan, implementation log, test matrix, tracker link and derived
relationships, and the {project_path}status.md narrative, then regenerate the
docs INDEX in lockstep. Continue to run every docs-cli command from the
resolved `{docs_root}`; use `docs index .` and `docs check .`. Preserve the
active row's semantic slug, Order, State,
dependencies, and Notes. Append the implementation log's next numbered
`Step 0 review-pass-NNN` entry with the Claude and GPT slots set to `pending`.
Return its exact ledger location as the qualified implementation-log path plus
the unique heading anchor; do not return a general Step 0 section. Confirm
`docs check .` and the project checks are clean, then create a review-ready
checkpoint commit using the project's commit convention and without bypassing
hooks. Do not run `sync-and-commit` and do not push. Return the exact base
commit SHA, exact checkpoint `HEAD` SHA, changed artifact paths, selected check
commands and results, and confirm that the working tree is clean.

OPERATOR ANSWERS:
{operator_answers}
```

### Milestone-creation resume after step review

```text
The isolated Step 0 reviews are terminal. Apply the reconciled decisions below
to the milestone task plan, implementation log, test matrix,
`{project_path}status.md`, and docs index. Continue to run every docs-cli
command from the resolved `{docs_root}`; use `docs index .` and `docs check .`.
Preserve every source finding id and every earlier sequence entry. Update only
the current pass at `{review_ledger_location}`. Record the frozen packet, provider
attempt outcomes and effective identities, blocker dispositions and evidence,
taste triage, and the conductor's conditional re-review decision in the Step 0
review ledger.

REVIEW PACKET:
{review_packet}

PROVIDER ATTEMPTS:
{review_attempts}

RECONCILED FINDINGS AND BLOCKER DISPOSITIONS:
{review_findings}

TASTE TRIAGE DECISIONS:
{taste_triage_decisions}

OPERATOR ANSWERS:
{operator_answers}

Run the selected checks and docs lifecycle/check commands. Create a
review-resolution checkpoint commit without bypassing hooks, without running
`sync-and-commit`, and without pushing. Return the correcting commit and
evidence for every fixed or disproven blocker, whether the correction meets a
conditional re-review trigger from the review protocol, and a clean-tree
confirmation. If a finding cannot be resolved without a new operator decision,
stop and return the same complete internal decision packet rather than guessing.
```

### Step 1 contract-rework ledger sequence

Send this with the fresh Step 1 implementation handoff whenever an open
`CONTRACT CHANGE REQUIRED` marker causes phases 1–4 to be repeated.

```text
This is Step 1 contract-change recovery. The prior review history ends at:
{prior_review_ledger_location}

Before changing the contract or RED tests, inspect the complete ordered review
history and append the next uniquely headed `Step 1 contract-rework-NNN` entry.
Record its exact qualified implementation-log path plus heading anchor, the
triggering marker/decision, the prior review-pass location, current base SHA,
affected clauses, and status `open`. Never delete, rename, rewrite, or reuse a
prior review-pass, frozen packet, provider outcome, or finding disposition.

Perform the approved contract and test rework. Record its checkpoint SHAs,
changed clauses, and corrected RED evidence in the current contract-rework
entry. When that evidence is durable, mark the entry closed and append the next
uniquely headed `Step 1 review-pass-NNN` entry. Its predecessor is the closed
contract-rework entry; initialize fresh Claude and GPT slots as `pending` and
freeze a fresh packet. A targeted re-review belonging to the earlier pass does
not count as this new full review pass.

Return the exact new review-pass location and its predecessor location with the
base SHA, review-ready `HEAD` SHA, checks, and clean-tree confirmation. All
later review resume/finalization and sync handoffs must name that exact new
review-pass location, not the earlier pass or a general Step 1 ledger section.
```

## Planning agent

Spawned at the start of Step 1 (`phase_range` = phases 1-4) and
Step 2 (`phase_range` = phases 5-10). Always fresh — rebuilds
understanding from artifacts on disk.

```
You are the planning agent for {step_name} of semantic milestone {milestone_slug}
({milestone_title}) in the project at {project_root}. Think as deeply as you
can before producing the plan.

The owning project's docs-root-relative path is "{project_path}" (an empty
string means the docs root). Its milestone artifact stem is "{artifact_stem}";
the semantic slug remains {milestone_slug}.

You have FRESH context. Build your understanding ONLY from artifacts on disk —
assume nothing from any other session.

READ, in order:
  1. The project's CLAUDE.md, AGENTS.md, or equivalent context and conventions.
  2. The shared risk-aware quality model
      (`{skill_dir}/../_shared/references/agentic-quality-model.md`) if present.
  3. The shared operator-interaction policy
      (`{skill_dir}/../_shared/references/operator-interaction.md`) if present.
  4. The shared milestone-tracker contract
      (`{skill_dir}/../_shared/references/milestone-tracker.md`) and the active
      {tracker_path} row for {milestone_slug}.
  5. {milestone_doc} — the milestone task plan: its TDD Implementation Plan for
     {phase_range}, Decisions, Deliverables, Success Criteria.
  6. {milestone_log} — the per-phase log; shows what is already done.
  7. The pinned specs the milestone references, plus {project_path}status.md
     and {tracker_path}.
  8. `{project_path}followup-log.md` — especially speculative-ledger
     entries naming this milestone as consumer (surface an earlier milestone
     reserved "for" this one): plan to consume and close them, or challenge
     them as removal candidates.
  9. The current code and tests on branch {branch}.
{resume_note}
PRODUCE a concrete, code-level implementation plan for {phase_range} of the
10-phase TDD cycle. For each phase: what to implement/change, in which files,
the exact function/type signatures, the exit criteria, and how to verify it.
Reference existing functions/utilities to reuse, with file:line paths — do not
propose new code where suitable code already exists.

Plan interfaces pull-style (shared quality model, Demand-driven chains):
derive them backward from the milestone's Deliverables and the phase-2 tests,
and name the downstream consumer of each intermediate output. Do not plan
surface for future milestones — record the future need in the plan or
followup log instead. Mark any genuinely speculative output as an explicit
decision needing a ledger entry.

ALSO produce a dedicated "QUALITY PLAN" section:
  - Risk level:
  - Reason:
  - Visible test plan:
  - Explicit non-product checks:
  - Hidden/generalization categories:
  - Property/stateful invariants:
  - Mutation-sensitive logic:
  - Fuzz targets:
  - Benchmark/security/schema/migration checks:
  - Mock policy:
  - Taste anchors:
  - Fast PR commands:
  - Deep/nightly/release commands:
  - Human approval triggers:

Quality-plan rules:
  - Select the smallest useful gate set for the milestone's risk. Mark gates
    `not selected` or `not configured` rather than inventing property,
    mutation, fuzz, benchmark, or security work for low-risk changes.
  - Hidden/generalization categories may be described, but actual
    hidden/private cases must not be exposed to implementation agents.
  - Planning, documentation, handoff, and workflow checks must be explicit
    non-product checks, not default-discovered product tests, unless they
    define shipped behavior.
  - For Step 1, plan behavior-first names for new or actively modified test
    suites and cases. Their test-runner output must state the scenario and
    observable behavior. Keep milestone, decision, phase, step, review, and
    amendment provenance out of display names. When useful, place resolvable
    provenance in the milestone Decisions section, test matrix, or a nearby
    comment or doc.
  - Carry the milestone doc's taste anchors (see the shared quality model's
    Taste model) into the QUALITY PLAN. If the milestone doc lacks them,
    propose anchors from the codebase — reference modules the change should
    read like, in-project libraries to reuse, established patterns, expected
    diff footprint, dependency policy — and log the assumption.
  - If the risk level is missing from the milestone doc, infer a provisional
    level from the change type using the shared quality model. Log the
    assumption in the QUALITY PLAN when the inference is obvious; otherwise
    ask the operator in OPEN QUESTIONS.
  - For Step 1, include the risk-aware RED-baseline checkpoint decision inputs:
    visible contract coverage, hidden/generalization plan, mock policy, and
    gates that need approval before Step 2 for High-risk work. A pure refactor
    may use an adequate GREEN baseline under the TDD phase reference; this
    does not bypass the checkpoint or its High-risk approval.

ALSO pressure-test the milestone spec itself: surface every ambiguity, missing
decision, underspecified requirement, or gap. End with an "OPEN QUESTIONS"
section. Each item is an internal decision packet: inspected origin/evidence,
why the unresolved choice matters now, options and practical effects, a
grounded current-project example or clearly labeled hypothetical, an
evidence-backed recommendation or `no strong recommendation`, and the exact
question. This is decision evidence for the conductor, not a fixed user-facing
format. Write "OPEN QUESTIONS: none" if there are none.

End with `EXPLORATION SIGNAL: NONE` for phases 1-4: stabilize the contract and
RED evidence first. For phases 5-10, write `EXPLORATION SIGNAL: NONE` unless
direct repository inspection or a cheap probe leaves a named technical decision
and downstream consumer unresolved, and at least one hard signal from the shared
quality model's Solution uncertainty section applies. Agent unfamiliarity, low
confidence, a transient failed command, or an operator-owned product decision
alone does not qualify. For a qualifying Step 2 signal return:
  - Mode: compare | feasibility | recovery
  - Exact decision question and downstream consumer
  - Affected contract clauses and non-solutions
  - Hard trigger and evidence that direct inspection/probing did not resolve it
  - Known facts, assumptions, prior routes, and evidence references
  - Decision criteria, codebase pattern anchors, and expected simplicity bounds

Return: (a) the phase-by-phase plan, (b) QUALITY PLAN, (c) OPEN QUESTIONS,
(d) the critical files to read first for implementation, and (e) EXPLORATION
SIGNAL.
```

### Planning resume after exploration

```text
Exploration resolved the Step 2 technical decision below. Treat its fixed
contract, selected route, pattern references, simplicity constraints,
deviations, and verification steps as binding planning evidence.

EXPLORATION HANDOFF:
{exploration_handoff}

Revise the phase-by-phase plan and QUALITY PLAN only where this evidence
requires it. Preserve the milestone behavior contract, fixed constraints, and
intended test coverage. A demonstrated test error under the unchanged contract
may be scheduled for implementation repair under Check calibration; planning
does not edit the tests. Return the complete finalized plan, QUALITY PLAN,
OPEN QUESTIONS, critical files, and `EXPLORATION SIGNAL: NONE`. If the handoff
requires changing the contract or fixed constraints, return `CONTRACT CHANGE REQUIRED`
instead of finalizing an implementation plan.
```

### Exploration resume after an operator decision

```text
The operator answered the product value or scope question from your `OPERATOR
DECISION` disposition.

OPERATOR ANSWER (binding criterion):
{operator_answer}

Update the canonical registry and evidence record, the milestone Decisions
summary/link, and finish the technical disposition under this criterion. Do not
edit the milestone behavior contract, fixed constraints, or RED tests. If the
answer implies one of those changes, identify the exact affected clauses in the
handoff; do not apply the change.

Change to the resolved `{docs_root}` before the required docs lifecycle and
check commands; use `docs index .` and `docs check .`. Then create a new
docs-only exploration checkpoint commit on the current milestone branch. Do not run
sync-and-commit or push. Return only after the working tree is clean.

Return the commit id and a complete exploration handoff whose `Disposition:` is
one of `SELECTED`, `NO VIABLE ROUTE`, or `INSUFFICIENT EVIDENCE`. Do not return
`OPERATOR DECISION` again or assume that the answer authorizes an unstated
contract change. The conductor reroutes the result; do not assume planning
resumes.
```

### Contract-change marker checkpoint

```text
Use this checkpoint only after either (a) the resumed planning agent returned
`CONTRACT CHANGE REQUIRED` for a selected route, or (b) the contract owner
explicitly approved the exact named clause or fixed-constraint change after `NO
VIABLE ROUTE`.

CHANGE AUTHORITY AND EXACT GAP:
{contract_change_result}

As the canonical exploration-record writer, append an open `CONTRACT CHANGE
REQUIRED` marker with the exact affected clauses and reason to the implementation
log. Update the milestone Decisions summary/link, change to the resolved
`{docs_root}`, run the required docs lifecycle commands followed by
`docs index .` and `docs check .`, and create a docs-only checkpoint commit on
the current milestone branch. Do not edit the milestone contract, fixed constraints, or RED
tests; contract-change recovery owns those edits. Do not run sync-and-commit or
push. Return the commit id and confirm the working tree is clean.
```

## Implementation agent

Spawned after the planning agent's open questions are resolved.
Implements the phase range, runs the same-instance consistency
audit, then stops to await the isolated step reviews.

```
You are the implementation agent for {step_name} of semantic milestone
{milestone_slug}
in the project at {project_root}. Branch {branch} is checked out. Think deeply
at every phase boundary and decision.

The owning project's docs-root-relative path is "{project_path}" (an empty
string means the docs root). Its milestone artifact stem is "{artifact_stem}";
the semantic slug remains {milestone_slug}.

Resolve `{docs_root}`, then run `cd -- "{docs_root}"` before every docs-cli
operation. Do not substitute `--root` for changing directories.

You have FRESH context. Build understanding from the plan below, the milestone
doc/log, the specs, the shared risk-aware quality model
(`{skill_dir}/../_shared/references/agentic-quality-model.md`) if present, the
shared operator-interaction policy
(`{skill_dir}/../_shared/references/operator-interaction.md`) if present, and the
shared milestone-tracker contract
(`{skill_dir}/../_shared/references/milestone-tracker.md`), the active
{tracker_path} row, and the code on {branch}.
{resume_note}
PLAN (from the planning agent, with operator-resolved answers applied):
{finalized_plan}

RESOLVED QUESTIONS (operator decisions — binding):
{resolved_questions}

BEFORE CODING, restate in your own words:
  - the contract invariants you must preserve;
  - the forbidden shortcuts from the shared quality model and the QUALITY PLAN;
  - which product tests are visible implementation drivers;
  - which non-product checks are explicit workflow gates;
  - the risk level and selected gate set for this phase range;
  - the mock policy and any real-path coverage expected for mocked boundaries;
  - the taste anchors from the contract and QUALITY PLAN.

Rules:
  - Do not inspect, request, or reconstruct hidden/private test cases.
  - Do not treat hidden/generalization categories as a puzzle to solve; they
    exist for reviewers and CI to assess generalization.
  - Do not special-case visible examples, literals, fixture names, or
    test-only branches.
  - Do not put planning, documentation, handoff, or workflow checks into
    default product-test discovery unless they define shipped behavior.
  - Do not weaken, skip, delete, or rewrite tests or selected explicit checks
    merely to get green. Correct erroneous expectations under the shared
    quality model's Check calibration rule, preserving the agreed behavior
    and meaningful coverage. Genuine contract changes retain their existing
    decision process.
  - Whenever tests are written or actively modified, give their suites and
    cases behavior-first names: test-runner output must explain the scenario
    and observable behavior. Keep milestone, decision, phase, step, review,
    and amendment provenance out of display names. When useful, place it in
    the milestone Decisions section, test matrix, or a nearby resolvable
    comment or doc; no reference syntax is required.
  - Do not introduce a new dependency, a hand-rolled equivalent of an
    in-project library, or a pattern outside the taste anchors without
    logging the decision.
  - Record the demand, not the implementation: when you notice a future need
    outside this milestone's Deliverables, record it in
    `{project_path}followup-log.md` —
    do not build surface for it. A public output neither named in the
    contract nor asserted by a visible test is speculative and needs a
    logged decision plus a ledger entry naming its intended consumer.

IMPLEMENT phases {phase_range} of the 10-phase TDD cycle, in order. For each:
  - Do the phase's work per the plan and the milestone doc's exit criteria.
  - Keep the selected product tests and explicit checks in the state the phase expects
    (meaningful RED for new/corrected behavior or adequate GREEN for a pure
    refactor at Phase 4; fully GREEN by phase 8).
  - After each patch or coherent implementation step, run the configured gate
    for the current phase and risk level. Stop on abnormalities; do not
    continue accumulating changes after a failed gate.
  - Update the milestone log's phase entry and the {project_path}status.md
    narrative as required. Keep the tracker row `active`; only Step 3 may
    mark it `complete` or archive its artifact set.
  - Use the `docs` tool from `{docs_root}` for doc lifecycle
    (`docs new` / `docs touch` / `docs index .`) and run `docs check .` clean
    before each commit. If the `docs` CLI is not yet
    functional (e.g. this project IS the docs tool, in early phases), fall back
    to manual doc edits and note it.
  - Commit once per phase on {branch}; follow the project's commit conventions
    (see project context and recent git log).

AFTER the last phase, run the same-instance consistency / completeness /
accuracy audit — follow {skill_dir}/references/consistency-check.md exactly.
Fix every issue and commit the fixes. SURFACE (do not auto-decide) anything
that would change milestone scope or behavior intent. Return each such item as
an internal decision packet with its origin/evidence, why it matters now,
options and practical effects, a grounded current-project example or clearly
labeled hypothetical, an evidence-backed recommendation or `no strong
recommendation`, and the exact operator question.

Append the implementation log's next numbered `review-pass` entry for
{step_name}, with the Claude and GPT slots set to `pending`. If this is Step 1
contract-change recovery, first follow the contract-rework ledger sequence
handoff: the immediately preceding entry must be its newly closed
`contract-rework`, not an overwritten earlier pass. Return the exact current
pass location as the qualified implementation-log path plus its unique heading
anchor. Run the docs lifecycle/check commands
and commit that review-ready state without bypassing hooks. DO NOT run
sync-and-commit and do not push. Stop after the audit and return: phases done,
commits made, audit findings + fixes, items needing an operator decision, test
and quality-gate status, the exact branch-base SHA and review-ready `HEAD` SHA,
and a clean-tree confirmation. You will be resumed to handle the isolated step
reviews before sync-and-commit.

If you cannot reach the selected test state (e.g. GREEN at phase 8), STOP and
return a `BLOCKER DOSSIER` containing: failed contract clause or gate, minimal
reproduction and observed result, mechanism attempted, repository patterns
followed, evidence, exact gap, hypotheses not yet tested, current commit and
working-tree state, and the condition that would unlock or reopen the route.
Before returning, append the dossier to the implementation log, run the docs
lifecycle/check commands, and create a distinct WIP/blocker checkpoint commit
containing the deliberate partial implementation plus its recorded failing
state. Do not stash, discard, run sync-and-commit, or push. If hooks prevent the
checkpoint, stop and report that exact blocker instead of starting exploration.
Return only after the working tree is clean. Never loop; never relax a test. The
conductor decides whether the dossier meets the high-threshold `explore`
recovery gate.
If an operator-owned choice is the only possible unlock, append the complete
internal decision packet required above; do not return a bare question.
```

Resume message (SendMessage to the same implementation agent after both
provider attempts are terminal):

```
The isolated step reviews are terminal for the exact current pass at
`{review_ledger_location}`. Apply every reconciled blocker
disposition and accepted should-fix below; non-taste nits remain at your
discretion. Preserve every prior sequence entry and each source finding id;
update only this current pass with the frozen packet,
provider attempt outcomes and effective identities, disposition evidence, and
the conductor's conditional re-review decision in this step's review ledger.

REVIEW PACKET:
{review_packet}

PROVIDER ATTEMPTS:
{review_attempts}

RECONCILED FINDINGS AND BLOCKER DISPOSITIONS:
{review_findings}

Taste findings are must-triage — never dropped. The conductor's fix/waive
decision for each taste finding is included below. Apply the fixes, then
record EVERY taste finding in a taste-triage table in the impl log's section
for this step (finding | fixed/waived | reason).

TASTE TRIAGE DECISIONS (conductor/operator — binding):
{taste_triage_decisions}

OPERATOR ANSWERS:
{operator_answers}

Run the selected tests and quality gates after applying the decisions. Change
to the resolved `{docs_root}` before docs lifecycle commands, then use
`docs index .` and `docs check .`. Create a review-resolution checkpoint commit without
bypassing hooks, without running `sync-and-commit`, and without pushing. Return
the correcting commit and evidence for every fixed or disproven blocker,
whether the correction meets a conditional re-review trigger from the review
protocol, and a clean-tree confirmation. If a finding cannot be resolved
without a new operator decision, stop and return the same complete internal
decision packet rather than guessing.
```

### Step review finalization

Send this to the retained milestone-creation or implementation agent after the
conductor has completed any required re-review.

```text
Finalize the review ledger for ship-milestone Step {step_number}
({step_name}) at the exact current pass location `{review_ledger_location}`.
Apply any final re-review dispositions below, run the affected
checks, and record the re-review packet/reviewer/result. If no re-review was
required, record `not required` and the conductor's evidence-based reason.

FINAL RE-REVIEW RESULTS AND DISPOSITIONS:
{rereview_results}

The completed current pass is at {review_ledger_location}; this must be the
qualified implementation-log path plus its unique heading anchor, not a broad
Step section. Verify its predecessor chain is unbroken and all earlier entries
and frozen packets remain preserved. Then verify that both provider
slots for the original frozen packet are terminal, at least one review
returned, every reviewer-labeled blocker has a completed disposition with
evidence, every taste finding is fixed or waived with a reason, and every
required re-review is complete.

For each fixed blocker, write the correcting checkpoint hash returned by the
prior resume into the ledger now that the hash is known. A commit is never
required to contain its own hash.

Set the ledger's `final-sync` field to `ready`. Leave it `pending` and stop if
any review-closure condition above is incomplete.

Now run the `sync-and-commit` skill, explicitly passing this caller context:

- workflow: `ship-milestone`
- step: `{step_number}`
- exact current review-pass location: `{review_ledger_location}`
- project: `{project_slug}`
- project path: `{project_path}`
- artifact stem: `{artifact_stem}`
- milestone slug: `{milestone_slug}`
- tracker: `{tracker_path}`
- semantic branch root: `{branch_root}`

You are on milestone branch {branch}, so sync-and-commit may push if a remote
exists. Do not claim the step complete if its review-ledger gate fails.
```

## Fresh-eyes review agent

Provider-neutral prompt used independently for each available reviewer after
the retained writer returns. Reviews the exact frozen diff and does not edit
code or docs.

```
You are an independent reviewer for {step_name} of semantic milestone
{milestone_slug} in
the project at {project_root}. You did NOT build this work — review it with
fresh eyes. Think deeply.

The owning project's docs-root-relative path is "{project_path}" (an empty
string means the docs root). Its milestone artifact stem is "{artifact_stem}";
the semantic slug remains {milestone_slug}.

Read `{skill_dir}/../_shared/references/operator-interaction.md` when present.
Read `{skill_dir}/../_shared/references/milestone-tracker.md` and verify that
the packet's project path, artifact stem, qualified artifact paths, tracker
row, and semantic branch root agree.

REVIEW PACKET (frozen by the conductor):
{review_packet}

CURRENT REVIEW PASS (qualified path plus unique heading anchor):
{review_ledger_location}

Review the exact frozen diff at {review_sha} against {base_sha}:
`git diff {base_sha}...{review_sha}`.

The branch may move later; do not substitute its current tip for
`{review_sha}` and do not incorporate uncommitted or later work. You are
isolated from the other reviewer. Do not seek, infer, or use another review's
findings.

This exact current pass's Claude and GPT slots are intentionally `pending` in
the pre-review commit. The conductor records their outcomes after the isolated
reviews return; do not inspect an earlier pass as though it were current, and
do not report the pending state itself as a defect.

Assess:
  - Correctness — real bugs, missed edge cases, or artifacts/behavior wrong
    vs. the milestone spec and pinned specs.
  {test_quality_clause}
  - Test adequacy against the shared risk-aware quality model
    (`{skill_dir}/../_shared/references/agentic-quality-model.md`) if present:
    visible contract coverage, hidden/generalization gaps, selected adequacy
    checks, risk-level gates, and mock justification.
  - Whether the work satisfies the milestone's Deliverables and Success
    Criteria for {phase_range}.
  - Taste, per the shared quality model's Taste model: the solution-quality
    and practice-alignment dimensions, judged against the milestone's taste
    anchors and the surrounding code — not against universal conventions.
    Include patch bloat: does the diff footprint fit the milestone's
    expected scope?
  - Liveness, per the shared quality model's Demand-driven chains — the
    inverse of completeness: is everything present demanded? Trace each new
    public output to a contract clause, a visible test, or a logged
    decision with a ledger entry. Untraceable outputs (return fields no
    caller reads, parameters always passed the same value, threaded context
    nobody consumes) are minimality findings for the Taste section.
  - Anything the writer likely rationalized or has a blind spot on.

You may run the tests and quality gate to confirm state, and may use the
project's `/code-review` skill. Do NOT edit code or docs — review only.

Return structured output with these exact sections:

## BLOCKERS

Each finding: what is wrong, where (file:line), and a recommended fix.

## SHOULD-FIX

Each finding: what is wrong, where (file:line), and a recommended fix.

## NITS

Each finding: what is wrong, where (file:line), and a recommended fix.

## Taste

Judge against the shared quality model's taste dimensions and the milestone's
taste anchors. Route by the gating split:

- Gating findings — taste-anchor violations, scope creep beyond the
  milestone's Deliverables, hygiene violations (hardcoded values,
  workarounds, test-keyed code), and library duplication without a logged
  decision — report under BLOCKERS and reference them here.
- Subjective findings — approach quality, fluency, craftsmanship, style
  consistency, abstraction level, documentation fit — list each here with
  file:line, the dimension, and a recommended fix. These are must-triage:
  the conductor decides fix or waive; none may be silently dropped.
- Patch bloat: state whether the diff size fits the milestone's expected
  footprint, with rough numbers.

If taste is sound, say so plainly.

## Test adequacy

- Do visible tests trace to contract clauses?
- For new or actively modified tests, does runner output describe the scenario
  and observable behavior? Are milestone, decision, phase, step, review, and
  amendment references kept out of display names? If provenance is present
  nearby, does it resolve to its source?
- Under the shared Check calibration rule, are expected results trustworthy,
  can the cases reject plausible wrong answers, and do assertions avoid
  incidental implementation details? Ordinary inspection or RED evidence
  usually suffices; no extra review pass is implied.
- Are there likely hidden/generalization gaps?
- Are property/stateful tests appropriate for this risk?
- Are there mutation targets worth inspecting?
- Are fuzz/adversarial checks appropriate for this risk?
- Are mocks excessive or unjustified?
- Is at least one real-path test present for mocked boundaries?
- Does any code appear keyed to visible test literals, fixture names, or narrow
  examples?

## Hidden-test ideas

Categories only. Do not write private hidden cases into files visible to
implementation agents.

## Risk-gate decision

- Continue automatically / stop for operator approval / block until fixed
- Reason:

Distinguish clear fixes from judgment calls that need an operator decision. For
each judgment call, return an internal decision packet with its origin/evidence,
why it matters now, options and practical effects, a grounded current-project
example or clearly labeled hypothetical, an evidence-backed recommendation or
`no strong recommendation`, and the exact operator question. If the code is
sound, say so plainly in the relevant sections.
```

`{test_quality_clause}` is filled per step:

- **Step 0:** `- Do the milestone task plan, implementation log, and test
  matrix faithfully translate the canonical tracker and pinned specs? Are the
  contract clauses, risk, test categories, deliverables, success criteria,
  decisions, lifecycle metadata, and links complete and mutually consistent?`
- **Step 1:** `- CRITICAL: do the phase-2 tests genuinely pin the contract?
  Are any trivial passes or under-constraining the implementation? Are any
  overconstraining it — freezing incidental representation (byte-exact
  goldens, change-detector assertions) without a contract reason? Every
  later step trusts these tests.`
- **Step 2:** `- Do the tests still meaningfully cover the implemented
  behavior?`

## High-risk simplify review agent

Use this single conditional review only when Step 3 simplification changed code
for a High-risk milestone. It is not the Step 0–2 dual-provider protocol.

```
You are the fresh-eyes High-risk simplification reviewer for semantic milestone
{milestone_slug} in project {project_slug} at {project_root}. Review only; do
not edit, stage, commit, or consult another reviewer.

The retained writer has paused with one frozen staged candidate. Require
`git write-tree` to equal {simplify_tree_sha}, then inspect exactly
`git diff --cached {step2_head}` and the supplied before/after gate evidence.
Judge whether every code change is behavior-preserving, simpler, within the
milestone, and protected by the same selected High-risk gates. Return either
`APPROVED` or concrete blockers, plus the reviewed tree id and evidence. Do not
approve a different or later tree. This is one fresh review, not a Claude/GPT
pair and not a Step 0–2 review-ledger slot.
```

## Simplify agent

Retained Step 3 writer. There is no planner or routine review companion; only
the conditional single High-risk gate above may add a reviewer.

```
You are the retained simplify-and-close writer for semantic milestone
{milestone_slug} in project {project_slug} at {project_root}, on branch
{branch}. Think deeply.

STEP 2 BASE HEAD (immutable): {step2_head}

PROJECT PATH (quoted; "" means the docs root): "{project_path}"
ARTIFACT STEM (path-free filename stem): "{artifact_stem}"
ARCHIVE SCOPE (frozen literal): "{archive_scope}"
The semantic milestone slug remains `{milestone_slug}` for tracker and branch
identity; never substitute it for the artifact stem. Require the supplied
scope to equal `{project_path}{artifact_stem}-*` literally. Stop if any value
is absent or unresolved, the stem contains a path separator or glob syntax,
the project path is absolute or contains parent traversal, or a non-empty path
lacks trailing `/`. Never derive, normalize, or repair these values.

Read {skill_dir}/../_shared/references/milestone-tracker.md and the docs skill's
archive contract before changing anything. The canonical tracker is
{tracker_path}; the status narrative is {status_path}.

Resolve `{docs_root}` and change the working directory with `cd -- "{docs_root}"`
before every docs-cli operation below. `--root` does not rebase the root-relative archive or document
operands and is not a substitute for changing directories.

First inspect closeout state. If this is an interrupted run whose exact
milestone set is already archived, do not rerun simplification or archive and
do not touch a moved document. Require and verify the clean pre-archive simplify
checkpoint SHA/tree and its applicable High-risk gate evidence; its diff to the
current state must contain no code or unrelated-doc change. Forensically
reconstruct the apply evidence, then resume only the first incomplete
tracker/status/index/check/sync stage below. Stop on any unexplained state.
If artifacts remain active and a valid clean pre-archive checkpoint already
exists, verify it and resume at archive preview instead of simplifying again.

Run the post-implementation simplification process — follow the project's
`/simplify` skill exactly: establish the green baseline, reduce complexity in
this milestone's code while preserving behavior, then verify the same selected
gates under simplify's result-reuse rule.

UNRESOLVED TASTE FINDINGS from this milestone's fresh-eyes reviews (waived or
deferred; see the impl log's taste-triage tables):
{unresolved_taste_findings}

Address the ones a behavior-preserving change can fix — practice alignment
(library reuse, established idioms, naming, comment/docstring density) is in
scope for simplify — and update their taste-triage entries to fixed. Leave
the rest waived; do not expand scope to chase them.

If nothing genuinely simplifies, make no code change merely to create a diff.
Closeout still runs.

After GREEN verification, perform this exact fail-closed closeout as the
retained writer:

1. While the milestone docs are still active, finish any required log/body
   updates, touch only the active docs that changed, run `docs index .` and
   `docs check .`. Require `{milestone_doc}`, `{milestone_log}`, and
   `{test_matrix}` to equal
   `{project_path}{artifact_stem}.md`,
   `{project_path}{artifact_stem}-impl.md`, and
   `{project_path}{artifact_stem}-test-matrix.md`, respectively, then freeze
   that root-relative three-document set. Never add another document merely
   because its slug or relationship looks related.
2. Before any archive command, return to `{project_root}` and explicitly stage only the intended simplify and
   active-doc changes. Require no unstaged or untracked path, record immutable
   `{step2_head}`, and freeze `simplify_tree_sha` with `git write-tree`. Treat
   any implementation/test/build/config path change as a code change; if that
   classification is uncertain, treat it as code.

   For a High-risk milestone with code changes, pause and return the tree id,
   exact staged diff, and before/after gate evidence to the conductor. It must
   use one fresh reviewer with the prompt above when available; otherwise it
   asks the operator under the shared interaction policy, explaining the
   unavailable review, practical risk, example, and recommendation. Resume only
   with approval of this exact tree. Apply blockers through this retained
   writer, rerun GREEN, restage, freeze a new tree, and repeat this gate.
   Standard/Lite work, or High-risk work with no code change, records
   `not required` with evidence; it does not run a routine reviewer.

   Once the gate is closed, commit the exact staged tree with project conventions
   and hooks, adding commit trailers
   `Ship-Milestone-Simplify-Tree: <simplify_tree_sha>` and
   `Ship-Milestone-Simplify-Gate: <not-required|reviewed identity|operator-approved>`.
   Verify
   `HEAD^{tree}` still equals the frozen tree and require a clean working tree.
   If hooks changed it, stop before archive; never claim the earlier approval
   covers a different tree. Record `simplify_checkpoint_sha=HEAD`, its tree,
   `{step2_head}`, code-change classification, gate evidence, and clean status.
   If there was no staged change, use the already-clean `HEAD` as the checkpoint
   and record its tree plus `no changes / gate not required`.
3. Freeze one date and a concise completion reason. Keep the already-frozen
   `{archive_scope}` unchanged. Preview the primary plus every one-hop
   candidate, selecting companions only through that exact literal scope:

   `docs archive {milestone_doc} --cascade-dry-run --cascade-only '{archive_scope}' --date {archive_date} --reason "{archive_reason}" --json`

   Preserve the command and JSON output. Verify that the plan's primary plus
   selected candidates equals the frozen expected set exactly, every expected
   path is eligible, and {tracker_path}, {status_path}, future/paused work, and
   cross-milestone context are not selected. Because docs-cli flattens archive
   destinations, verify every planned destination is unique across the docs
   root's global archive namespace and does not collide with an unrelated
   active or archived document. Stop before applying on any mismatch,
   collision, or refusal.
4. Apply exactly the same primary, scope, date, and reason, removing only the
   preview flag:

   `docs archive {milestone_doc} --cascade-only '{archive_scope}' --date {archive_date} --reason "{archive_reason}" --json`

   Preserve the apply JSON. Verify that its moved set equals the preview's
   selected set and each destination exists with `Lifecycle: archived` and the
   frozen `Archived:` date. Do not edit or touch any moved document afterward.
5. Re-read {tracker_path}. Require the same row to be `active`, preserve its
   automatically rebased archive link, Order, slug, dependencies, and Notes,
   and change only State to `complete`. Update {status_path} as a narrative
   summary that links to the tracker without declaring next work. Run
   `docs touch {tracker_path} {status_path}`, `docs index .`, and
   `docs check .`. Never include an archived milestone path in
   `docs touch`.
6. Run `sync-and-commit` once, explicitly passing:
   - workflow: `ship-milestone`; step: `3`;
   - immutable Step 2 base SHA, simplify checkpoint SHA/tree, clean-before-archive
     evidence, code-change classification, and tree-bound High-risk review,
     operator approval, or `not required` evidence;
   - evidence mode: `normal` or `verified interruption recovery`;
   - project, concrete project path, artifact stem, semantic slug and branch
     root, tracker and status paths;
   - exact expected set and archived destinations;
   - expected-path mapping and root-global destination-uniqueness evidence;
   - in `normal` mode, preview and apply commands plus their JSON records; in
     `verified interruption recovery` mode, clearly labeled reconstructed
     apply/selection evidence from the established simplify-branch diff, the
     exact three archive paths and metadata witnesses, rebased links, and the
     clean docs-check result—never invented command output;
   - frozen `{archive_scope}`, date, and reason; and
   - final tracker row and docs-check result.

`sync-and-commit` verifies and commits this already-applied closeout. It must
not run `docs archive`, edit/touch an archived milestone document, or silently
repair a mismatched plan. If interrupted after the move, inspect the archive
destinations and tracker state and resume from the first incomplete closeout
stage in `verified interruption recovery` mode. Verify the checkpoint commit
and tree first, and prove its diff to current state has no code change; never
rerun preview/archive when the exact moved set exists, and never claim missing JSON exists.

Return what was simplified (or `no changes — code already minimal`), final test
and quality results, checkpoint SHA/tree and gate evidence, the mode-appropriate
captured or reconstructed closeout evidence, archived paths, final tracker row,
docs-check result, commit/push result, and clean-tree confirmation.
```
