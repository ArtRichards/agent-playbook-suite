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
6. [Approach-family search](#approach-family-search)
7. [Disposable probes](#disposable-probes)
8. [Synthesis and adversarial gate](#synthesis-and-adversarial-gate)
9. [Stopping and dispositions](#stopping-and-dispositions)
10. [Handoff and resume](#handoff-and-resume)
11. [Host fallback](#host-fallback)

## Roles and boundaries

The coordinator owns the decision contract, canonical record, route
deduplication, probe authorization, synthesis, and final disposition. Approach
scouts and challengers return evidence; they do not edit the canonical record.
Keep one writer for that record so parallel work cannot overwrite decisions.

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

If any line is missing, continue ordinary inspection or ask the owning workflow
to stabilize its inputs. Do not invoke the full portfolio process.

An issue is sufficiently novel only when inspection shows that no local or
authoritative pattern covers the core mechanism, or that the combination of
fixed constraints makes the known pattern inapplicable. Cite that inspection.

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
contract issue for its owner.

## Repository pattern baseline

Inspect the relevant surface before proposing routes. Scope the inspection to
the decision, but do not infer a project pattern from one convenient file.
Record:

```markdown
## Pattern baseline

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

### Route registry

Assign stable route IDs and deduplicate by underlying mechanism, not phrasing.

```markdown
## Route registry

| Route | Mechanism | Clauses | Pattern references | Evidence | Status | Exact gap / rejection | Reopen trigger |
|---|---|---|---|---|---|---|---|
| R-01 | ... | C-01, C-02 | ... | E-01 | exploring | ... | ... |
```

Allowed statuses:

- `exploring` - evidence collection is active;
- `viable` - no known failure remains, but final gates are incomplete;
- `selected` - final disposition chose this route;
- `rejected` - evidence shows the route cannot meet the fixed contract or is
  dominated by a simpler route;
- `blocked` - an external condition currently prevents a decision or probe;
- `deferred` - currently lower decision value; reconsider only if leading
  routes fail or constraints change.

Never rewrite `rejected` as though it was not tried. Reopen a rejected or
blocked route only for one of these named triggers:

1. new evidence that directly answers its recorded gap;
2. a materially different mechanism within the family;
3. a changed contract or fixed constraint; or
4. resolution of the external condition that blocked it.

Record the trigger before reopening.

### Evidence ledger

Separate observation from interpretation:

```markdown
## Evidence ledger

| Evidence | Route / clause | Source or command | Observation | Interpretation | Limitations |
|---|---|---|---|---|---|
| E-01 | R-01 / C-02 | `command` or `path:line` | ... | ... | ... |
```

Prefer repository artifacts, reproducible commands, probe output, and primary
technical sources. Mark estimates and reasoned inferences explicitly. A route
is not viable merely because a scout found no problem.

After every scout or probe return, update the registry and ledger immediately.
At each synthesis point append: new evidence, invalidated assumptions, route
status changes, exact remaining gaps, and the next decision-changing action.
This is the resume state.

## Approach-family search

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
unless a reopen trigger appears. After the first returns, synthesize before
launching more work. Open another family or probe only when its possible result
could change the final disposition.

## Disposable probes

Inspection comes first. Probe only an uncertainty that can distinguish routes
or determine clause feasibility.

Before execution, write a probe card:

```markdown
### P-01: <question>

- Route / clauses: R-01 / C-02
- Hypothesis: ...
- Success observation: ...
- Failure observation: ...
- Environment: <temporary copy, disposable worktree, container, scratch DB>
- Allowed mutations: ...
- Cleanup: ...
- Decision effect: <how each outcome changes route status>
```

Use the least invasive environment that reproduces the relevant constraint.
Prefer a temporary directory or disposable worktree pinned to the inspected
revision. Keep credentials, network access, privileged actions, and destructive
operations subject to the host and project policies already in force.

Probes may contain throwaway code, fixtures, or dependencies inside the
isolated environment. They must not alter the production worktree, lockfiles,
project configuration, persistent services, or shared data. Do not turn probe
code into a merge candidate; the implementation owner should build from the
recorded constraints and evidence.

After execution, record exact commands, revision/environment, observed output,
limitations, cleanup result, and the evidence-ledger row. A benchmark without a
representative workload or a compatibility check against only a mock is weak
evidence; label it accordingly.

## Synthesis and adversarial gate

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
registry, and evidence ledger to a fresh challenger. Require it to search for:

- a missed existing pattern or simpler mechanism;
- unsupported clause coverage;
- unnecessary abstractions, interfaces, dependencies, or layers;
- hidden migration or operational work;
- evidence that does not reproduce the claimed constraint;
- a near miss listed under non-solutions; and
- a counterexample that would invalidate the route.

Every challenge must cite evidence or be marked `reasoned-only` with a concrete
verification path. Triage each finding as fixed in the recommendation,
disproved with evidence, or an exact remaining gap. Material changes to the
leading route require another focused challenge of the changed candidate.

## Stopping and dispositions

After each synthesis, ask: **Could one more bounded investigation change the
disposition?** Continue only when the answer is yes and the expected decision
value justifies the cost.

Stop when one of these is true:

1. a route passes every contract, pattern-preservation, simplicity, evidence,
   and fresh-challenge gate;
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
evidenced for handoff, not guaranteed truth.

### `OPERATOR DECISION`

Use when technical evidence has reduced the issue to a product value or
priority. State the viable options, consequences, reversibility, cost of later
change, any recommendation already implied by the charter or project criteria,
and the smallest question the operator must answer. After the answer, bind it
as a criterion and finish the technical disposition.

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
- Disposition:
- Selected route or exact gap:
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
the repository revision and fixed constraints have not changed. Revalidate
stale evidence or record a changed-constraint reopen trigger; do not restart
the search from memory.

## Host fallback

Use the host's available worker mechanism without changing the protocol:

- In Codex, use collaboration workers such as `spawn_agent` and collect their
  returned reports; use follow-up messaging only to request missing evidence.
- In Claude Code, use the available Agent/Task mechanism with fresh worker
  context and collect reports before synthesis.
- On another host with parallel workers, use its equivalent general-purpose or
  research workers and keep canonical-record writes with the coordinator.
- With no worker support, run family passes sequentially. Start each from the
  frozen contract and pattern baseline, keep prior family conclusions out of
  the pass, store its raw return, then synthesize only after the independent
  passes finish.

If isolation facilities or authoritative sources are unavailable, continue
with read-only evidence where useful and return `INSUFFICIENT EVIDENCE` when a
required gate cannot be supported. Missing host capability is not permission to
weaken the contract or fabricate evidence.
