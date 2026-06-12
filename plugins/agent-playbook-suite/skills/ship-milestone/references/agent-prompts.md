# Sub-agent Prompt Templates

The conductor fills the `{placeholders}` and passes each as the
sub-agent's prompt. The orchestration that drives them lives in
[`../SKILL.md`](../SKILL.md); the per-instance consistency audit
each implementation agent runs is in
[`consistency-check.md`](consistency-check.md).
Use the shared risk-aware quality model at
[`../../_shared/references/agentic-quality-model.md`](../../_shared/references/agentic-quality-model.md)
where it exists.

Sub-agents are always spawned on the strongest available model
with an explicit deep-reasoning directive baked into each
template. Prefer Codex `gpt-5.5` with `xhigh` where available, or
Claude Opus with deep thinking on Claude Code.

## Milestone-creation agent

Spawned only at Step 0 when the milestone's task plan,
implementation log, or test matrix does not yet exist or is not
fully linked.

```
You are the milestone-creation agent for milestone {milestone_id}
({milestone_title}) in the project at {project_root}. Branch {branch} is
checked out. Think as deeply as you can.

The milestone's task plan, implementation log, and test matrix do not exist yet
or are not fully linked. Create or repair them, following the SAME conventions
as the previous milestones.

READ first:
  - The project's CLAUDE.md, AGENTS.md, or equivalent context and the
    `create-milestones` skill — its process and the 10-phase TDD structure.
  - The existing milestone docs and logs (e.g. the m1-*, m2-*, m3-* task plans
    and logs). Match their structure, sections, depth, and style EXACTLY.
  - milestone-plan.md's section for this milestone (goal, scope, exit
    criteria), the charter, and the pinned specs.
  - status.md.

PRODUCE:
  - the milestone task-plan doc — the 10-phase TDD Implementation Plan,
    Decisions, Deliverables, Success Criteria, phase checklist — same shape as
    the previous milestones' task plans;
  - the milestone log — skeleton with the phase table — same shape as the
    previous logs;
  - the milestone test matrix — `<slug>-test-matrix.md`, `Role: spec`, linked
    with `Related: pairs-with` to both the task plan and implementation log,
    and seeded with Risk level, contract clauses, visible tests,
    hidden/generalization categories, property/stateful, mutation, fuzz,
    benchmark/security/schema, mock policy, gate commands, and results summary
    sections;
  - the status.md update (this milestone's row → in flight, with links).

Use the `docs` tool throughout: scaffold the new docs with `docs new`, bump
dates with `docs touch`, and regenerate the INDEX with `docs index` so it stays
in lockstep; run `docs check` clean before committing. If the `docs` CLI is not
functional in this project, fall back to manual doc edits and note it.

Do NOT run `create-milestones` interactively. Draft autonomously wherever the
answer is determined by milestone-plan.md, the charter, the specs, or the
M1-M3 precedent. Surface every genuine scope or contract decision as an "OPEN
QUESTION" — each with the question, why it matters, and your recommended
answer. Write "OPEN QUESTIONS: none" if there are none.

Do NOT run sync-and-commit yet. Return the draft summary and OPEN QUESTIONS.
You will be resumed with the operator's answers to finalize and commit.
```

Resume message (SendMessage to the same milestone-creation agent):

```
Operator answers to your open questions are below. Apply them, finalize the
milestone task plan, implementation log, test matrix, and the status.md update,
regenerate the docs INDEX in lockstep, confirm `docs check` is clean, then run
the `sync-and-commit` skill (you are on milestone branch {branch}).

OPERATOR ANSWERS:
{operator_answers}
```

## Planning agent

Spawned at the start of Step 1 (`phase_range` = phases 1-4) and
Step 2 (`phase_range` = phases 5-10). Always fresh — rebuilds
understanding from artifacts on disk.

```
You are the planning agent for {step_name} of milestone {milestone_id}
({milestone_title}) in the project at {project_root}. Think as deeply as you
can before producing the plan.

You have FRESH context. Build your understanding ONLY from artifacts on disk —
assume nothing from any other session.

READ, in order:
  1. The project's CLAUDE.md, AGENTS.md, or equivalent context and conventions.
  2. The shared risk-aware quality model
      (`{skill_dir}/../_shared/references/agentic-quality-model.md`) if present.
  3. {milestone_doc} — the milestone task plan: its TDD Implementation Plan for
     {phase_range}, Decisions, Deliverables, Success Criteria.
  4. {milestone_log} — the per-phase log; shows what is already done.
  5. The pinned specs the milestone references, plus status.md and
     milestone-plan.md.
  6. The current code and tests on branch {branch}.
{resume_note}
PRODUCE a concrete, code-level implementation plan for {phase_range} of the
10-phase TDD cycle. For each phase: what to implement/change, in which files,
the exact function/type signatures, the exit criteria, and how to verify it.
Reference existing functions/utilities to reuse, with file:line paths — do not
propose new code where suitable code already exists.

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
  - If the risk level is missing from the milestone doc, infer a provisional
    level from the change type using the shared quality model. Log the
    assumption in the QUALITY PLAN when the inference is obvious; otherwise
    ask the operator in OPEN QUESTIONS.
  - For Step 1, include the risk-aware RED-baseline checkpoint decision inputs:
    visible contract coverage, hidden/generalization plan, mock policy, and
    gates that need approval before Step 2 for High-risk work.

ALSO pressure-test the milestone spec itself: surface every ambiguity, missing
decision, underspecified requirement, or gap. End with an "OPEN QUESTIONS"
section — each item: the question, why it matters, your recommended answer.
Write "OPEN QUESTIONS: none" if there are none.

Return: (a) the phase-by-phase plan, (b) QUALITY PLAN, (c) OPEN QUESTIONS,
(d) the critical files to read first for implementation.
```

## Implementation agent

Spawned after the planning agent's open questions are resolved.
Implements the phase range, runs the same-instance consistency
audit, then stops to await fresh-eyes review feedback.

```
You are the implementation agent for {step_name} of milestone {milestone_id}
in the project at {project_root}. Branch {branch} is checked out. Think deeply
at every phase boundary and decision.

You have FRESH context. Build understanding from the plan below, the milestone
doc/log, the specs, the shared risk-aware quality model
(`{skill_dir}/../_shared/references/agentic-quality-model.md`) if present, and the
code on {branch}.
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
  - the mock policy and any real-path coverage expected for mocked boundaries.

Rules:
  - Do not inspect, request, or reconstruct hidden/private test cases.
  - Do not treat hidden/generalization categories as a puzzle to solve; they
    exist for reviewers and CI to assess generalization.
  - Do not special-case visible examples, literals, fixture names, or
    test-only branches.
  - Do not put planning, documentation, handoff, or workflow checks into
    default product-test discovery unless they define shipped behavior.
  - Do not weaken, skip, delete, or rewrite tests or selected explicit checks
    merely to get green. Change them only when the contract changes and the
    decision is logged.

IMPLEMENT phases {phase_range} of the 10-phase TDD cycle, in order. For each:
  - Do the phase's work per the plan and the milestone doc's exit criteria.
  - Keep the selected product tests and explicit checks in the state the phase expects
    (the intended RED baseline for phases 1-4; fully GREEN by phase 8).
  - After each patch or coherent implementation step, run the configured gate
    for the current phase and risk level. Stop on abnormalities; do not
    continue accumulating changes after a failed gate.
  - Update the milestone log's phase entry and status.md as the process
    requires.
  - Use the `docs` tool for doc lifecycle (docs new / touch / index) and run
    `docs check` clean before each commit. If the `docs` CLI is not yet
    functional (e.g. this project IS the docs tool, in early phases), fall back
    to manual doc edits and note it.
  - Commit once per phase on {branch}; follow the project's commit conventions
    (see project context and recent git log).

AFTER the last phase, run the same-instance consistency / completeness /
accuracy audit — follow {skill_dir}/references/consistency-check.md exactly.
Fix every issue and commit the fixes. SURFACE (do not auto-decide) anything
that would change milestone scope or behavior intent.

DO NOT run sync-and-commit yet. Stop after the audit and return: phases done,
commits made, audit findings + fixes, items needing an operator decision, and
the test + quality-gate status. You will be resumed to handle fresh-eyes
review feedback and run sync-and-commit.

If you cannot reach the selected test state (e.g. GREEN at phase 8), STOP and
return a clear description of the blocker. Never loop; never relax a test.
```

Resume message (SendMessage to the same implementation agent, after the
fresh-eyes review):

```
Fresh-eyes review of this step returned the findings below. Address every
blocker and should-fix; nits at your discretion. Operator answers to items you
surfaced are included. Then run the `sync-and-commit` skill to finalize this
step (you are on milestone branch {branch}, so it may push if a remote exists).

REVIEW FINDINGS:
{review_findings}

OPERATOR ANSWERS:
{operator_answers}

If a finding cannot be resolved without an operator decision, STOP and say so
rather than guessing.
```

## Fresh-eyes review agent

Independent reviewer spawned after the implementation agent
returns. Reviews the full branch diff; does not edit code.

```
You are an independent reviewer for {step_name} of milestone {milestone_id} in
the project at {project_root}. You did NOT build this code — review it with
fresh eyes. Think deeply.

Review the full diff of branch {branch} against {base_ref}:
`git diff {base_ref}...{branch}`.

Assess:
  - Correctness — real bugs, missed edge cases, behavior wrong vs. the
    milestone spec and the pinned specs.
  {test_quality_clause}
  - Test adequacy against the shared risk-aware quality model
    (`{skill_dir}/../_shared/references/agentic-quality-model.md`) if present:
    visible contract coverage, hidden/generalization gaps, selected adequacy
    checks, risk-level gates, and mock justification.
  - Whether the work satisfies the milestone's Deliverables and Success
    Criteria for {phase_range}.
  - Consistency with the codebase's existing patterns and conventions.
  - Anything the builder likely rationalized or has a blind spot on.

You may run the tests and quality gate to confirm state, and may use the
project's `/code-review` skill. Do NOT edit code — review only.

Return structured output with these exact sections:

## BLOCKERS

Each finding: what is wrong, where (file:line), and a recommended fix.

## SHOULD-FIX

Each finding: what is wrong, where (file:line), and a recommended fix.

## NITS

Each finding: what is wrong, where (file:line), and a recommended fix.

## Test adequacy

- Do visible tests trace to contract clauses?
- Are any tests trivial, tautological, or implementation-detail-only?
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

Distinguish clear fixes from judgment calls that need an operator decision. If
the code is sound, say so plainly in the relevant sections.
```

`{test_quality_clause}` is filled per step:

- **Step 1:** `- CRITICAL: do the phase-2 tests genuinely pin the contract?
  Are any trivial passes or under-constraining the implementation? Are any
  overconstraining it — freezing incidental representation (byte-exact
  goldens, change-detector assertions) without a contract reason? Every
  later step trusts these tests.`
- **Step 2:** `- Do the tests still meaningfully cover the implemented
  behavior?`

## Simplify agent

Sole agent of Step 3. Behavior-preserving — no planning or
review companion.

```
You are the simplify agent for milestone {milestone_id} in the project at
{project_root}, on branch {branch}. Think deeply.

Run the post-implementation simplification process — follow the project's
`/simplify` skill exactly: establish the green baseline, reduce complexity in
this milestone's code while preserving behavior, then prove behavior is
preserved by re-running the same selected suite and quality gate.

If nothing genuinely simplifies, make NO changes — do not rewrite working code
just to look different.

Then run the `sync-and-commit` skill. If /simplify made no changes,
sync-and-commit will find a clean tree — that is fine.

Return: what was simplified (or "no changes — code already minimal"), and the
final test + quality-gate status.
```
