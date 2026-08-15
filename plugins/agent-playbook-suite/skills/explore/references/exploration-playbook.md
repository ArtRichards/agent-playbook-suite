# Exploration Playbook

Use this playbook to compare technical mechanisms, establish feasibility, or
recover from a blocked implementation route without turning exploration into
production development.

## Contents

1. [Roles and boundaries](#roles-and-boundaries)
2. [Activation and mode selection](#activation-and-mode-selection)
3. [Decision contract](#decision-contract)
4. [Repository pattern baseline](#repository-pattern-baseline)
5. [Canonical record](#canonical-record)
6. [Evidence waves and route frontier](#evidence-waves-and-route-frontier)
7. [Disposable probes](#disposable-probes)
8. [Synthesis, advisor, and adversarial gate](#synthesis-advisor-and-adversarial-gate)
9. [Stopping and dispositions](#stopping-and-dispositions)
10. [Handoff and resume](#handoff-and-resume)
11. [Host fallback](#host-fallback)

## Roles and boundaries

The coordinator owns the small serial core: decision-contract and shared-
baseline changes, the canonical record, route deduplication, probe
authorization, wave synthesis, and final disposition. Evidence scouts,
approach scouts, probe executors, advisors, and challengers return bounded
reports; they do not edit the canonical record. Keep one writer for that record
so parallel work cannot overwrite decisions.

Organize independent work in evidence waves separated by short synthesis
barriers. Every worker in a wave starts from the same frozen checkpoint. Do not
feed one independent return into another worker or continually resynthesize as
returns arrive. Stage the raw returns, then merge observations and update route
state once at the wave boundary. Run work sequentially only when a real evidence
dependency requires it, the work cannot be safely isolated, or the host cannot
isolate workers. Read-only does not automatically mean independent: external
queries may share quotas, credentials, confidential data, production load, or
cost budgets.

Exploration may:

- read project code, tests, configuration, dependency manifests, history, and
  authoritative technical documentation;
- compare mechanisms and estimate the affected code surface;
- run read-only checks and bounded, reversible probes;
- write decision state to the owning planning or log artifact; and
- recommend concrete implementation constraints and verification steps.

Exploration must not:

- invent or expand product requirements;
- decide an operator-owned product decision when project criteria do not
  settle it;
- edit production code or produce a patch intended to merge;
- add persistent dependencies, migrations, generated files, or configuration;
- continue a failed route without new evidence; or
- equate agent agreement, popularity, or elegance with evidence.

If the behavior contract is unstable, return to the owning planning workflow.
If the question is how users should work with the product, use `use-cases`. If
the route is already known and the task is implementation or ordinary
debugging, continue with the milestone workflow.

## Activation and mode selection

### Automatic invocation

Use the three-part gate from `SKILL.md`. Record it in this compact form:

```text
Decision: <one technical question>
Consumer: <artifact, phase, or implementation owner>
Cheap check: <inspection/check performed and result>
Hard signal: <specific unresolved assumption, missing precedent, route fork,
or invalidating evidence>
Mode: compare | feasibility | recovery
```

For automatic invocation, if any line is missing, continue ordinary inspection
or ask the owning workflow to stabilize its inputs. Do not invoke the full
portfolio process.

An issue is sufficiently novel only when inspection shows that no local or
authoritative pattern covers the core mechanism, or that the combination of
fixed constraints makes the known pattern inapplicable. Cite that inspection.

### Direct invocation

Honor a direct request by running the same cheap check and making the decision
contract explicit. If that check resolves the question, return its evidence
without manufacturing a route portfolio. Otherwise continue when the contract
and at least one hard signal are present; ask the operator only for an input that
cannot be discovered and would materially change the decision.

### Mode choice

- Choose `compare` when two or more mechanisms could satisfy the same contract
  and the decision depends on evidence about fit or consequences.
- Choose `feasibility` when one critical unknown determines whether a candidate
  can satisfy the contract. Include a comparison baseline so a successful
  probe does not automatically prove the probed design is simplest.
- Choose `recovery` when an attempted route has a minimal reproduction or other
  concrete invalidating evidence. Preserve the failed route and its evidence;
  do not erase it when alternatives are opened.

Allow the mode to change. A failed feasibility probe can become recovery; a
recovery search can become compare when multiple alternatives survive.

## Decision contract

Freeze a bounded contract before assigning scouts or spending on probes. Use
stable IDs so every route and evidence row can cite the requirement it affects.

```markdown
## Decision contract: <D-001 short name>

- Mode: compare | feasibility | recovery
- Decision: <one falsifiable technical question>
- Downstream consumer: <who or what will act on the result>
- Completion claim: <exact user- or system-observable result the implementation
  must eventually provide>
- Preconditions / supported domain: <where the claim must hold>
- Fixed constraints: <compatibility, policy, performance, dependency, rollout>
- Non-solutions: <outcomes that look useful but do not satisfy the decision>
- Probe boundary: <allowed time, environment, access, and mutation>
- Operator-owned choices: <known value decisions, or none>

| Clause | Required behavior or property | Evidence needed to select a route |
|---|---|---|
| C-01 | ... | ... |
| C-02 | ... | ... |
```

Good clauses distinguish routes and can be checked. Include failure and
recovery behavior, compatibility boundaries, operational constraints, and the
relevant use-case or milestone contract references. Examples of non-solutions
include a happy-path-only prototype, one supported adapter when the contract
requires several, a mock-only integration result, or moving the hard behavior
behind an unused interface.

Do not let an approach scout redefine this contract. If evidence reveals that
a clause is impossible or underspecified, pause route selection and record the
contract issue for its owner. Scouts report such defects explicitly; do not
collapse an unmeasurable or contradictory clause into an ordinary evidence
gap. Still return exactly one disposition: use `OPERATOR DECISION` when the
owner must supply a missing acceptance criterion or value choice; use
`NO VIABLE ROUTE` only when evidence shows the fixed clauses eliminate every
route; and use `INSUFFICIENT EVIDENCE` when the clause is well formed but the
evidence or calibration required to evaluate it is inaccessible.

When route selection depends on an empirical evaluator or probe oracle, verify
its calibration and representative domain before scheduling probes that consume
its result. The contract may freeze the intended success criterion while the
baseline records whether the available evaluator validly measures it.

## Repository pattern baseline

Inspect enough of the relevant surface to freeze a trustworthy shared baseline
before proposing routes. Scope the inspection to the decision, but do not infer
a project pattern from one convenient file. Keep this serial baseline to facts
every worker needs; assign separable repository areas, authoritative-source
checks, or verifier audits to independent evidence scouts and merge their
observations at the first wave boundary. Record:

```markdown
## Pattern baseline

Repository revision / evidence as-of: <revision, date, or immutable snapshot>
Freshness boundary: <changes that require revalidation>

| Concern | Established reference | Constraint on the solution |
|---|---|---|
| Module ownership | `path/to/module` | ... |
| Interfaces / data flow | `path/to/interface` | ... |
| Errors / recovery | `path/to/tests-or-code` | ... |
| Configuration | `path/to/config` | ... |
| Dependencies | `manifest` | ... |
| Tests / fixtures | `path/to/tests` | ... |
| Naming / observability | `path/to/reference` | ... |

Simplest viable baseline: <smallest change shape that might satisfy all clauses>
Expected affected surface: <modules, interfaces, data, tests, operations>
Unresolved pattern gap: <none, or exact missing precedent>
```

Also read the project instruction file and relevant architecture or decision
records. Inspect history only when it could explain why a pattern exists or why
a prior mechanism was rejected.

Pattern preservation is behavioral and structural, not cosmetic. Prefer the
project's existing libraries, abstractions, ownership boundaries, error model,
configuration, logging, naming, and test strategy. Treat a new abstraction as
a cost even when it reduces a few local lines.

The baseline is not a miniature solution analysis. Stop extending it when the
contract, local reference points, simplest plausible shape, and unresolved
evidence questions are clear enough for independent work. A scout may report a
missed pattern, but only the coordinator may add it to the shared baseline at a
synthesis barrier.

## Canonical record

### Artifact routing

Write exploration state where its downstream consumer already works:

| Lifecycle moment | Canonical record |
|---|---|
| Foundation architecture | `options-comparison.md`; place durable decisions in the existing decision log if the project uses one. |
| Milestone comparison or feasibility | The milestone implementation log is canonical; keep the behavior contract authoritative in the milestone plan and put only the disposition plus a link in its Decisions section. |
| Failed-route recovery | The affected milestone implementation log, beside the failure evidence and next implementation step; other artifacts link to it. |
| Standalone, cross-cutting, or multi-session | A separate `idea` document created with `docs new idea <topic>-exploration` only when no owning artifact is suitable. |

In a docs-managed tree, create a new document only through the `docs` skill and
`docs new`; update lifecycle and index through docs-cli. Outside a docs-managed
tree, use the project's existing decision-record convention. For a short direct
exploration with no durable project artifact, keep the registry and evidence
current in response-local scratch state before each synthesis; the final
response is the canonical record. Do not create a persistent artifact merely to
satisfy the record shape.

Scale the recorded form to the decision. A single-clause question with one
surviving family may keep the contract, registry, ledger, checkpoint, and
focused challenge in a few compact lines; every gate still applies. Promote
response-local state to a durable owning or standalone artifact as soon as the
exploration crosses a session or context boundary, opens a second wave, the host
cannot reliably retain worker returns, or it otherwise becomes unsafe to
reconstruct from the final response alone.

### Route registry

Assign stable route IDs and deduplicate by underlying mechanism, not phrasing.

```markdown
## Route registry

| Route | Mechanism | Clauses | Pattern references | Evidence | Status | Exact gap / rejection | Wake / reopen trigger |
|---|---|---|---|---|---|---|---|
| R-01 | ... | C-01, C-02 | ... | E-01 | exploring | ... | ... |
```

Allowed statuses:

- `exploring` - evidence collection is active;
- `viable` - the route could still satisfy every clause and no known failure
  remains, but final gates are incomplete;
- `selected` - final disposition chose this route;
- `rejected` - evidence shows the route cannot meet the fixed contract or is
  dominated by a simpler route;
- `blocked` - an external condition currently prevents a decision or probe;
- `deferred` - the route could still satisfy every clause but currently has
  lower decision value; record the evidence, leader failure, advisor direction,
  blocker resolution, or constraint change that would wake it.

Never rewrite `rejected` as though it was not tried. Reopen a rejected or
blocked route only for one of these named triggers:

1. new evidence that directly answers its recorded gap;
2. evidence that its recorded rejection generalized a variant failure beyond
   what the family-level mechanism supports;
3. a changed contract or fixed constraint; or
4. resolution of the external condition that blocked it.

Record the trigger before reopening. Register a materially different mechanism
as a new route family instead of using it to reopen an old one.

Each `R-xx` identifies one mechanism family. Record an attempted variant as an
optional `V-xx` on its probe card and evidence rows, not as another route. A
failed parameter, adapter, prototype shape, or incomplete instance rejects only
that variant unless the evidence falsifies the family's defining mechanism.
Only then may the result change the route's status. Preserve a near
miss as `viable` or `deferred` only when it still plausibly satisfies the fixed
contract, contributes distinct clause coverage or a plausible complementary
mechanism, and has a named next decision-changing action. Preserve reusable
evidence in the ledger even when its route is rejected; evidence usefulness
never makes a non-viable route viable. Do not keep ornamental alternatives.

At each synthesis barrier, append a compact frontier checkpoint:

```markdown
## Frontier checkpoint: <W-01>

- Current decision bottleneck: <one exact uncertainty or gate>
- Leading route, if any: <R-xx and why it currently leads>
- Independent survivors: <R-xx and the distinct reason each remains>
- Decision-relevant near misses: <R-xx, learning, and next action>
- Retired this wave: <R-xx and mechanism-level or variant-level reason>
- Latest evidence change: <what became known, invalid, or narrower>
- Next wave: <bounded questions or probes, or stop>
```

The frontier has no required size. Keep as many routes as have distinct
decision value and no more.

### Evidence ledger

Separate observation from interpretation:

```markdown
## Evidence ledger

| Evidence | Route / variant / clause | Source or command | Observation | Interpretation | Limitations |
|---|---|---|---|---|---|
| E-01 | R-01 / V-01 / C-02 | `command` or `path:line` | ... | ... | ... |
```

Prefer repository artifacts, reproducible commands, probe output, and primary
technical sources. Mark estimates and reasoned inferences explicitly. A route
is not viable merely because a scout found no problem.

Before opening a wave, record its frozen inputs and assigned questions. Give
independent workers an immutable packet containing the contract, shared
baseline, assignment, checkpoint ID, and only the role-specific operational
context needed to execute safely: inspected revision, project instructions,
bounded source surface, or approved probe environment and safety limits as
applicable. Do not give them the live canonical-record path, rankings, or other
workers' returns. Stage raw returns in coordinator-owned scratch or the host
mailbox, not in a location later workers are instructed to read. At the wave
boundary, merge candidate rows into the ledger once and append: new evidence,
invalidated assumptions, route status changes, the frontier checkpoint, exact
remaining gaps, and the next decision-changing action. This checkpoint is the
resume state. If a wave is interrupted, synthesize the completed returns
explicitly and list the missing assignments rather than silently treating them
as negative evidence.

## Evidence waves and route frontier

### Wave protocol

Use a wave when two or more bounded tasks can proceed from the same contract
and baseline without consuming one another's conclusions:

1. Freeze the checkpoint and name the current decision bottleneck.
2. Split work by independent evidence question, mechanism family, or isolated
   probe; avoid duplicate assignments disguised by wording.
3. For every assignment, check evidence dependencies, mutable state, external
   side effects, shared repository or service state, rate limits, scarce
   environments, credentials and data disclosure, and cost budget.
4. Run assignments concurrently only when those checks and host policy allow it.
5. Stage raw returns without declaring a winner.
6. Close the wave with one evidence merge, frontier update, and inquiry
   checkpoint.

Use a direct bounded pass rather than a wave when only one task can change the
decision. Do not parallelize a true dependency. If one result determines the
next question, close the current wave first and open another from the new
checkpoint. The coordinator may close a wave early when completed evidence
changes the bottleneck or a worker becomes a straggler; cancel or defer the
unfinished assignments explicitly and never treat them as negative evidence.

### Independent evidence work

Use evidence scouts for route-independent questions such as the applicable
repository convention, the validity of the evaluator or success oracle, a
compatibility boundary, or an authoritative technical claim. Give each scout
one falsifiable question and prohibit route selection. This converts unknown
unknowns into explicit gaps without making the coordinator perform every read
serially.

If downstream probes depend on the evaluator or oracle, complete and synthesize
that validation before opening the dependent probe wave. Do not co-schedule an
oracle audit with work whose route status would rely on that oracle.

### Approach-family search

Derive families from mechanisms that could satisfy the same contract: reuse an
existing extension point, adapt an adjacent internal pattern, use an already-
approved dependency capability, change data flow within current boundaries, or
make the smallest evidenced deviation. Do not use generic labels such as
"simple," "robust," or "hybrid" as families.

Keep early work independent:

- Give each scout the same decision contract and pattern baseline, plus one
  family to investigate.
- Do not reveal a favored route, other scouts' reasoning, or the desired
  conclusion.
- Ask for concrete clause coverage, repository references, expected diff
  footprint, new concepts, failure modes, and evidence gaps.
- Let scouts return reports; keep the coordinator as the sole canonical-record
  writer.
- Run families in parallel when the host supports safe independent workers.

Use an adaptive portfolio. Explore no fixed number of families. Do not spawn
near-duplicate scouts, and do not continue a family after its decisive failure
unless a reopen trigger appears. Do not force a structural family to beat the
current leader on its first variant; ask whether a negative result falsifies
the mechanism or only its present instance. Preserve a decision-relevant near
miss long enough to run its smallest informative follow-up.

Open a combined route only when evidence suggests two families cover distinct,
complementary gaps and the resulting concepts and footprint can still pass the
simplicity gate. Do not use combination as a way to rescue unrelated rejected
ideas. After a wave returns, synthesize before launching more work. Open another
family or probe only when its possible result could change the final
disposition.

## Disposable probes

Inspection comes first. Probe only an uncertainty that can distinguish routes
or determine clause feasibility.

Before execution, write a probe card:

```markdown
### P-01: <question>

- Route / clauses: R-01 / C-02
- Variant: V-01 or none
- Hypothesis: ...
- Success observation: ...
- Failure observation: ...
- Oracle validity: <calibration, representative domain, and known blind spots>
- Environment: <temporary copy, disposable worktree, container, scratch DB>
- Allowed mutations: ...
- Cleanup: <including parent-repository or external state created by setup>
- Candidate decision effect: <how each outcome could affect clauses; the
  coordinator assigns route status after synthesis>
```

Use the least invasive environment that reproduces the relevant constraint.
Prefer a temporary directory or disposable worktree pinned to the inspected
revision. Keep credentials, network access, privileged actions, and destructive
operations subject to the host and project policies already in force.

A Git worktree is not fully isolated: it mutates the parent repository's
metadata and shares its object store and refs. Use a separate temporary copy or
clone when probes run concurrently or policy requires zero parent-repository
mutation. When a worktree is sufficient, declare those metadata mutations,
serialize conflicting Git operations, remove only the exact disposable
worktree path, and verify that only its registration disappeared. Treat
repository-wide pruning as a separate policy-controlled action outside the
probe cleanup.

Run probes in the same wave only when they use isolated environments, share no
mutable resource or rate limit, and neither probe's question depends on the
other's result. Otherwise sequence them across synthesis barriers.

Probes may contain throwaway code, fixtures, or dependencies inside the
isolated environment. They must not alter the production worktree, lockfiles,
project configuration, persistent services, or shared data. Do not turn probe
code into a merge candidate; the implementation owner should build from the
recorded constraints and evidence.

After execution, record exact commands, revision/environment, observed output,
limitations, cleanup result including parent or external state, and the
evidence-ledger row. A benchmark without a representative workload or a
compatibility check against only a mock is weak evidence; label it accordingly.
Only the coordinator changes route status after merging the result and its
limitations.

## Synthesis, advisor, and adversarial gate

Compare viable routes in one table:

```markdown
| Route | Clause coverage | Pattern fit | Expected footprint | New concepts | Operational / migration effects | Evidence gaps |
|---|---|---|---|---|---|---|
| R-01 | ... | ... | ... | ... | ... | ... |
```

For each route, answer:

1. Which exact repository patterns does it reuse?
2. What files, modules, interfaces, data, configuration, and tests should the
   implementation affect?
3. Is there a smaller route that satisfies every clause?
4. Which new dependency, abstraction, public interface, layer, or persistent
   concept does it add, and why is each necessary?
5. What failure, rollback, compatibility, resource, concurrency, or operations
   behavior follows from the mechanism?
6. What evidence would falsify the route?

### Inquiry checkpoint

At every synthesis barrier, ask these questions in order:

1. What exact bottleneck or missing fact currently prevents the disposition?
2. What did the latest evidence change in the contract map, frontier, or
   expected implementation footprint?
3. Which information are we still missing, and what is the cheapest reliable
   way to obtain it?
4. Does the proposed next work test a new falsifiable claim, or merely vary the
   same idea without a reason?
5. Is the evaluator or probe oracle measuring the required behavior across the
   relevant domain, or could it be rewarding a non-solution?
6. Is there a missed repository pattern, authoritative source, adjacent-domain
   mechanism, or independent combination that could change the frontier?
7. Which bounded action now has the highest expected decision value, and what
   result would make the exploration stop?

Record the answers that change route state or the next action; do not turn the
questions themselves into a large diary.

### Advisor escalation

Declare the frontier stagnant when proposed next work repeats an existing
mechanism without a new falsifiable claim, the latest returns add no relevant
evidence or sharper gap, or the team keeps tuning the current leader while a
named clause-coverage gap suggests missing mechanisms or evidence. Tie that gap
to a first falsifiable check. First restate the bottleneck and check that weak
instrumentation, an unstable contract, or a stale or anchored coordinator
context is not the actual cause. When context is stale, checkpoint the canonical
record and resume from it in a fresh context before escalating.

Then give a fresh, high-capability advisor the decision contract, shared
baseline, an unranked and reordered view of surviving mechanisms and evidence,
rejected routes with reasons, and the exact stagnation checkpoint without the
leader label. Ask it for better questions, genuinely orthogonal mechanism
families, missed evidence sources, or a new way to discriminate the frontier.
Do not identify a desired answer. After the gate fires, read and instantiate
the [Fresh Frontier Advisor Prompt](fresh-frontier-advisor.md). Before sending
any project material to an external CLI or service, confirm that host, network,
confidentiality, data-disclosure, credential, and cost policy already authorize
that exact packet; otherwise use an authorized in-host worker or the sealed
sequential fallback.
The advisor:

- generates hypotheses and evidence paths; it does not select a route;
- may recommend authoritative external reading but does not turn popularity or
  model recall into evidence;
- must distinguish a new mechanism from a local variant or renamed route;
- must state expected clause impact, simplicity cost, and the first falsifier;
  and
- must return `NO MATERIAL NEW DIRECTION` instead of manufacturing novelty.

The coordinator deduplicates the return, verifies its claims, and opens only
the questions whose answers could change the disposition. Advisor authority is
never evidence. Carry the advisor's source-status label onto every route or
evidence lead it originates; keep `reasoned-only` visible until independent
evidence supports it. Escalate at most once for an unchanged bottleneck; repeat
only after new evidence or a materially changed checkpoint. Keep this role
separate from the final adversarial challenger: the advisor expands or reframes
the frontier; the challenger tries to disprove the actual leading route.

### Deviation test

A deviation from an established pattern passes only when the record contains:

- the pattern and concrete reference that would normally apply;
- the contract clause it cannot satisfy;
- evidence of that mismatch, not merely preference;
- the smallest sufficient deviation and expected diff footprint;
- migration, compatibility, and maintenance consequences; and
- a verification step for the implementation owner.

Do not add a special approval checkpoint. Honor any existing High-risk,
security, dependency, or operator policy gate.

### Fresh challenge

Give the actual leading route, decision contract, pattern baseline, route
registry, and evidence ledger to a genuinely fresh, policy-authorized
challenger. Before sending that packet to an external CLI or service, apply
the same host, network, confidentiality, data-disclosure, credential, and cost
authorization gate used for the advisor. Require the challenger to search for:

- a missed existing pattern or simpler mechanism;
- unsupported clause coverage;
- unnecessary abstractions, interfaces, dependencies, or layers;
- hidden migration or operational work;
- evidence that does not reproduce the claimed constraint;
- a listed non-solution; and
- a counterexample that would invalidate the route.

Every challenge must cite evidence or be marked `reasoned-only` with a concrete
verification path. Triage each finding as fixed in the recommendation,
disproved with evidence, or an exact remaining gap. Material changes to the
leading route require another focused challenge of the changed candidate. A
challenger `FAIL` categorically blocks `SELECTED`: adding future implementation
constraints to the handoff is not remediation and cannot turn it into a pass.
Resolve or change the recommendation, challenge that final candidate again,
and require the latest result to be `PASS`; otherwise use an unresolved
disposition.

## Stopping and dispositions

After each synthesis, ask: **Could one more bounded action change the
disposition?** Continue only when the answer is yes and the expected decision
value justifies the cost.

Stop when one of these is true:

1. a route passes every contract, pattern-preservation, simplicity, evidence,
   and fresh-challenge gate, the latest challenge of the final candidate is
   `PASS`, and no unresolved surviving route could materially change the
   selection;
2. evidence eliminates every route within the fixed contract and constraints;
3. all remaining routes depend on an external blocker;
4. viable routes are separated only by an operator-owned product decision that
   existing project criteria do not resolve; or
5. additional probes have diminishing decision value and cannot justify a
   selection.

Use exactly one disposition:

### `SELECTED`

Name the selected route and cite clause-by-clause evidence. Include the pattern
references, expected footprint, necessary new concepts, evidenced deviations,
implementation constraints, verification plan, residual assumptions, and why
the strongest alternative was not selected. `SELECTED` means sufficiently
evidenced for handoff, not guaranteed truth. It requires a `PASS` from the
latest fresh challenge of the final candidate. Do not select while any
`viable` or `deferred` survivor retains an unresolved difference that could
materially change the choice; resolve, reject, or report that gap instead.

### `OPERATOR DECISION`

Use when technical evidence has reduced the issue to a product value or
priority, or when the contract owner must clarify an underspecified acceptance
criterion before routes can be evaluated. State the viable options or contract
defect, consequences, reversibility, cost of later change, any recommendation
already implied by the charter or project criteria, and the smallest question
the operator must answer. Do not invent the missing criterion. After the
answer, bind it as a criterion through the owning workflow and finish the
technical disposition.

### `NO VIABLE ROUTE`

Use only when evidence eliminates all routes under the current contract and
constraints. List each route's decisive failure and identify which clause or
constraint must change before search could resume. Return that decision to the
contract owner; exploration does not edit the contract or treat a proposed
change as approved. Resume under changed terms only after the owner explicitly
approves them and the owning workflow records the change.

### `INSUFFICIENT EVIDENCE`

Use when selection cannot be justified because required evidence is
inaccessible, all remaining probes are externally blocked, or further work has
diminishing value. Name the exact evidence gap, why it distinguishes the
routes, what was tried, and the event or input that could resolve it.

Do not report a route preference as `SELECTED`, and do not label missing effort
as impossibility. A no viable route result requires elimination evidence for
every recorded family.

## Handoff and resume

Return a compact handoff packet:

```markdown
## Exploration handoff

- Decision / mode:
- Canonical record:
- Inspected revision / evidence as-of:
- Freshness boundary:
- Disposition:
- Selected route or exact gap:
- Strongest alternative / why it was not selected:
- Contract evidence: <C-xx -> E-xx>
- Pattern references reused:
- Necessary deviations:
- Expected implementation footprint:
- Implementation constraints and non-solutions:
- Verification steps:
- Residual assumptions / blockers:
- Next owner and action:
```

The implementation owner should be able to act without inheriting the scouts'
full conversation. Link the canonical record rather than copying stale route
state into multiple artifacts.

On resume, read the decision contract, latest synthesis checkpoint, route
registry, evidence ledger, and probe cards before doing new work. Verify that
the inspected revision, freshness boundary, and fixed constraints still hold.
Revalidate stale evidence or record a changed-constraint reopen trigger; do not
restart the search from memory.

## Host fallback

Use the host's available worker mechanism without changing the protocol:

- In Codex, use collaboration workers such as `spawn_agent` and collect their
  returned reports; use follow-up messaging only to request missing evidence.
  For the fresh advisor and challenger, use `gpt-5.6-sol` with `xhigh`
  reasoning when available; record any substituted effective model. Use the
  advisor only at the escalation point, not as a routine vote.
- In Claude Code, use the available Agent/Task mechanism with fresh worker
  context and collect reports before synthesis. For the fresh advisor and
  challenger, use the `opus` alias with `xhigh`; the alias tracks the newest
  supported Opus model. A headless advisor or challenger call is also
  acceptable only when policy authorizes sending that role's exact packet to
  the service; it receives the same frozen packet and cannot edit the
  canonical record. Record any substituted effective model.
- On another host with parallel workers, use its equivalent general-purpose or
  research workers and keep canonical-record writes with the coordinator.
- With no worker support, run evidence and family passes sequentially. Start
  each from the frozen contract and pattern baseline, keep prior pass
  conclusions out of the pass, complete and seal its raw return before the next
  pass, then synthesize only after the independent passes finish. If prior
  conclusions cannot be excluded, label the independence limitation in the
  evidence ledger. A sealed coordinator pass is not a fresh context: do not run
  an advisor or challenger prompt that claims otherwise. Use a genuinely fresh,
  policy-authorized context for those roles. If none is available, record the
  missing gate and return `INSUFFICIENT EVIDENCE` rather than `SELECTED`.

If isolation facilities or authoritative sources are unavailable, continue
with read-only evidence where useful and return `INSUFFICIENT EVIDENCE` when a
required gate cannot be supported. Missing host capability is not permission to
weaken the contract or fabricate evidence.
