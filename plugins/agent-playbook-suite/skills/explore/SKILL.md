---
name: explore
description: "Investigate clear solution uncertainty or novel technical implementation decisions without doing production implementation. Use directly, or from project-foundation, create-milestones, or ship-milestone, only after repository inspection or a cheap probe leaves a named decision unresolved and at least one hard signal remains: an acceptance-critical assumption is unverified, no established applicable pattern fits, materially different mechanisms remain plausible without selection evidence, or the selected route was blocked or invalidated. Supports architecture comparison, feasibility checks, and failed-route recovery; not for use-case discovery, routine inspection or debugging, requirements invention, or implementation."
---

# explore

Resolve one bounded technical decision with repository evidence. Return an
implementation-ready route only when it preserves established patterns,
satisfies the decision contract, and is the simplest evidenced option.
Otherwise return the exact unresolved gap. Keep the irreducibly serial work of
framing, synthesis, and disposition small; organize the rest as independent
evidence work from shared checkpoints.

## Read first

- Read [`references/exploration-playbook.md`](references/exploration-playbook.md)
  in full before running an exploration.
- Read [`references/agent-prompts.md`](references/agent-prompts.md) before
  assigning independent workers or using the sealed sequential fallback. A
  single direct coordinator pass does not require it.
- Read [`references/fresh-frontier-advisor.md`](references/fresh-frontier-advisor.md)
  only after the playbook's Advisor escalation gate fires.
- Read the repository's `AGENTS.md`, `CLAUDE.md`, or equivalent instructions
  before inspecting or probing the project.

## Activation gate

Do not add a full exploration merely because the implementation is unfamiliar
or an agent has low confidence. For automatic invocation, require all three:

1. Name the technical decision and its downstream consumer.
2. Inspect the relevant repository surface, authoritative material, or run one
   cheap reversible check; confirm that this does not resolve the decision.
3. Record at least one hard signal:
   - an acceptance-critical technical assumption remains unverified;
   - no established local or authoritative pattern applies to the core
     mechanism;
   - materially different mechanisms remain plausible, and evidence does not
     yet distinguish them; or
   - concrete evidence blocked or invalidated the selected route.

Treat novelty as an evidenced absence of an applicable precedent, not as a
label. An ordinary failed command, operator-owned product decision, broad desire
to "research options," or routine debugging is not a hard signal by itself.
On direct invocation, perform the same cheap check; if it resolves the issue,
return that evidence without manufacturing an exploration.

## Modes

Select one initial mode; change it only when new evidence warrants it:

| Mode | Use it to |
|---|---|
| `compare` | Select among materially different architecture or implementation mechanisms. |
| `feasibility` | Test an acceptance-critical unknown before committing to a route. |
| `recovery` | Replace or reopen a route after concrete evidence blocks or invalidates it. |

`explore` evaluates technical routes. Send user-workflow discovery to
`use-cases`, stabilize behavior and scope with the owning planning skill, and
leave production implementation to the milestone workflow.

## Invariants

1. **Contract before search.** Fix the decision question, stable behavior
   clauses, success oracle, constraints, non-solutions, consumer, and probe
   boundary before comparing routes.
2. **Repository patterns are the default.** Inventory relevant modules,
   interfaces, tests, dependencies, naming, error handling, configuration, and
   operational conventions. Do not select a route without concrete reference
   points.
3. **Simplicity gate.** Compare every candidate with the smallest viable
   implementation. Justify each new dependency, abstraction, public interface,
   layer, or persistent concept.
4. **Deviation needs evidence, not a new approval.** Depart from an established
   pattern only when evidence shows it cannot satisfy a contract clause. Choose
   the smallest sufficient deviation and record its rationale and footprint.
   Preserve any approvals already required by project policy; invent none.
5. **Keep the serial core small.** Serialize at least contract and baseline
   changes, probe authorization, evidence merge, route status, and final
   disposition, plus any work that cannot be safely isolated. Batch independent
   reconnaissance, route analysis, and safe probes from the same checkpoint; do
   not make one independent pass wait for or inherit another pass's conclusion.
6. **Preserve a frontier, not one incumbent.** Keep materially different viable
   and decision-relevant near-miss families alive until evidence retires them.
   Do not reject a mechanism family because one implementation variant failed,
   and do not preserve a route without a named learning or next action.
7. **Evidence outranks consensus.** Keep approach families initially
   independent, deduplicate them by mechanism, and require artifacts, commands,
   observations, or authoritative sources for material claims.
8. **Probes are disposable.** Permit small prototypes, benchmarks, or
   compatibility checks only in an isolated temporary copy or disposable
   worktree with a predeclared oracle. Do not edit production code, create
   mergeable implementation, or persist dependency changes.
9. **One canonical record.** Keep the full registry and evidence in one existing
   owning artifact by default; other artifacts carry only a compact disposition
   and link. Create a separate exploration document only for a standalone or
   genuinely multi-session investigation.
10. **Fail closed.** Do not return `SELECTED` when contract, pattern,
   simplicity, evidence, or adversarial checks are incomplete. The latest
   fresh challenge of the final candidate must return `PASS`; a `FAIL` or
   `INSUFFICIENT EVIDENCE` remains blocking until remediation and another
   challenge. State the exact gap instead of projecting certainty.

## Procedure

1. **Frame the decision.** Choose the mode and write the exact decision
   contract with stable clause IDs.
2. **Build the minimum shared baseline.** Inspect the relevant code and record
   the applicable patterns, simplest viable baseline, unresolved uncertainty,
   and evidence sources every worker needs. Delegate separable reconnaissance
   rather than expanding the coordinator's serial analysis.
3. **Open the record and checkpoint.** Initialize the route registry, evidence
   ledger, current frontier, and inquiry checkpoint in the owning artifact.
4. **Run a bounded pass or evidence wave.** Use one direct pass for one
   decision-changing task; use a wave for independent reconnaissance,
   materially different mechanism families, or safely isolated probes from the
   same checkpoint. State every probe's oracle and isolation plan first. Check
   dependencies, side effects, shared resources, rate limits, credentials, data
   disclosure, and cost before parallel work. Do not create alternatives to meet
   a quota or leak a favored answer across passes.
5. **Merge once and update the frontier.** After the pass or wave, separate
   observation from interpretation, deduplicate mechanisms, compare clause
   coverage, pattern fit, footprint, new concepts, operational consequences,
   and evidence quality, then record route status changes and the next decision
   bottleneck.
6. **Ask the checkpoint questions.** Identify what is actually limiting the
   disposition, what information is missing, what the latest evidence changed,
   whether proposed work is only another local variant, whether the oracle could
   reward a non-solution, whether a missed pattern or adjacent-domain mechanism
   could change the frontier, and which bounded action has the highest decision
   value.
7. **Escalate for fresh ideas only when warranted.** If the frontier stagnates
   or a named clause gap suggests missing mechanisms or evidence, give a fresh
   advisor an unranked frozen packet and the exact bottleneck. Use its return to
   open testable questions or genuinely different families, never as selection
   evidence. Escalate once per unchanged bottleneck; require new evidence or a
   materially changed checkpoint before repeating it. Subject the eventual
   leading route to a separate fresh adversarial pattern-and-simplicity review;
   only that review's latest `PASS` can authorize `SELECTED`.
8. **Stop and hand off.** Continue only while another bounded action could
   change the disposition. Persist the final registry, evidence, deviations,
   remaining assumptions, and exact next action for the downstream consumer.

Follow the schemas, route lifecycle, reopen rules, artifact routing, and stop
conditions in the exploration playbook.

## Dispositions

- `SELECTED` - one route passes every contract, pattern-preservation,
  simplicity, evidence, and adversarial gate, its final candidate has a latest
  fresh-challenge result of `PASS`, and no unresolved surviving route could
  materially change the selection.
- `OPERATOR DECISION` - viable routes remain separated by a value or product
  priority that existing project criteria do not resolve, or the contract owner
  must clarify an underspecified acceptance criterion before routes can be
  evaluated. Supply technical consequences and the smallest decision required;
  do not choose silently or invent the missing criterion.
- `NO VIABLE ROUTE` - evidence eliminates every route within the fixed
  contract and constraints. Identify the clauses and constraints responsible
  and return them to the contract owner. Do not reopen search under changed
  terms until that owner explicitly approves the change.
- `INSUFFICIENT EVIDENCE` - an external blocker, inaccessible evidence, or
  diminishing-value boundary prevents a justified selection. Name the exact
  missing evidence, why it matters, and what could resolve it.

Never disguise a partial answer as `SELECTED`. Exploration recommends a route;
it does not implement it. Report no viable route only when evidence eliminates
every route under the fixed contract and constraints.
