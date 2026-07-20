---
name: explore
description: "Investigate clear solution uncertainty or novel technical implementation decisions without doing production implementation. Use directly, or from project-foundation, create-milestones, or ship-milestone, only after repository inspection or a cheap probe leaves a named decision unresolved and at least one hard signal remains: an acceptance-critical assumption is unverified, no established applicable pattern fits, materially different mechanisms remain plausible without selection evidence, or the selected route was blocked or invalidated. Supports architecture comparison, feasibility checks, and failed-route recovery; not for use-case discovery, routine inspection or debugging, requirements invention, or implementation."
---

# explore

Resolve one bounded technical decision with repository evidence. Return an
implementation-ready route only when it preserves established patterns,
satisfies the decision contract, and is the simplest evidenced option.
Otherwise return the exact unresolved gap.

## Read first

- Read [`references/exploration-playbook.md`](references/exploration-playbook.md)
  in full before running an exploration.
- When independent workers are available, also read
  [`references/agent-prompts.md`](references/agent-prompts.md) and use its
  host-neutral prompts. Without workers, run the same passes sequentially
  with separate notes and an evidence firewall between approach families.
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
5. **Evidence outranks consensus.** Keep approach families initially
   independent, deduplicate them by mechanism, and require artifacts, commands,
   observations, or authoritative sources for material claims.
6. **Probes are disposable.** Permit small prototypes, benchmarks, or
   compatibility checks only in an isolated temporary copy or disposable
   worktree with a predeclared oracle. Do not edit production code, create
   mergeable implementation, or persist dependency changes.
7. **One canonical record.** Keep the full registry and evidence in one existing
   owning artifact by default; other artifacts carry only a compact disposition
   and link. Create a separate exploration document only for a standalone or
   genuinely multi-session investigation.
8. **Fail closed.** Do not return `SELECTED` when contract, pattern,
   simplicity, evidence, or adversarial checks are incomplete. State the exact
   gap instead of projecting certainty.

## Procedure

1. **Frame the decision.** Choose the mode and write the exact decision
   contract with stable clause IDs.
2. **Build the baseline.** Inspect the relevant code and record the applicable
   patterns, simplest viable baseline, and unresolved uncertainty.
3. **Open the record.** Initialize the route registry and evidence ledger in
   the owning artifact. Record each synthesis checkpoint immediately.
4. **Explore independent families.** Assign or run materially different
   mechanisms without sharing a favored answer. Do not create alternatives to
   meet a quota.
5. **Probe only decision-changing gaps.** State the oracle and isolation plan
   first; record the command, artifact, observation, and limitation afterward.
6. **Synthesize and challenge.** Compare clause coverage, pattern fit, diff
   footprint, new concepts, operational consequences, and evidence quality.
   Subject the actual leading route to a fresh adversarial pattern-and-
   simplicity review.
7. **Stop and hand off.** Continue only while another bounded probe could
   change the disposition. Persist the final registry, evidence, deviations,
   remaining assumptions, and exact next action for the downstream consumer.

Follow the schemas, route lifecycle, reopen rules, artifact routing, and stop
conditions in the exploration playbook.

## Dispositions

- `SELECTED` - one route passes every contract, pattern-preservation,
  simplicity, evidence, and adversarial gate.
- `OPERATOR DECISION` - viable routes remain separated by a value or product
  priority that existing project criteria do not resolve. Supply technical
  consequences and the smallest decision required; do not choose silently.
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
