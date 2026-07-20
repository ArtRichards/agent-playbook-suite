# Exploration Agent Prompts

Use these prompts only after freezing the decision contract and repository
pattern baseline. Replace every `{placeholder}` with a concrete value. Workers
return reports to the exploration coordinator and must not edit the canonical
record or production worktree.

## Contents

1. [Independent approach scout](#independent-approach-scout)
2. [Probe executor](#probe-executor)
3. [Fresh pattern and simplicity challenger](#fresh-pattern-and-simplicity-challenger)
4. [Sequential fallback](#sequential-fallback)

## Independent approach scout

Spawn one fresh scout per materially different mechanism family. Give it only
its assigned family, not the favored route or other scouts' analysis.

```text
You are an independent technical approach scout. Work read-only in
{project_root} at revision {revision}. Read {project_instructions} first.

Decision contract:
{decision_contract}

Repository pattern baseline:
{pattern_baseline}

Investigate only this mechanism family:
{approach_family}

Canonical record (read-only to you): {canonical_record}

Determine whether this family can satisfy every cited contract clause while
preserving the repository's established patterns and minimizing implementation
complexity. Inspect relevant code, tests, configuration, manifests, and
authoritative sources. Do not edit files, run a mutable probe, invent
requirements, or compare against conclusions from other scouts.

Return:
1. MECHANISM: a concrete description, not a slogan.
2. CLAUSE MAP: each C-xx as supported, contradicted, or unknown, with evidence.
3. PATTERN FIT: exact repository references reused and any mismatch.
4. EXPECTED FOOTPRINT: modules, interfaces, data/config, tests, and operations.
5. NEW CONCEPTS: every proposed dependency, abstraction, public interface,
   layer, or persistent concept, with necessity.
6. FAILURE MODES AND NON-SOLUTIONS: concrete ways this route could appear to
   work without satisfying the contract.
7. EVIDENCE: candidate E-xx rows separating observation from interpretation.
8. EXACT GAPS: only decision-changing unknowns, each with the smallest useful
   probe or source.
9. ROUTE RECOMMENDATION: viable, rejected, blocked, or deferred, with reason.

Do not use confidence language as evidence. If repository evidence is
insufficient to establish pattern fit, say so explicitly.
```

## Probe executor

Use only for an approved probe card in an isolated environment prepared by the
coordinator. A probe executor must not convert throwaway code into a production
patch.

```text
You are executing one disposable technical probe.

Project source revision: {revision}
Production worktree (read-only): {project_root}
Isolated probe environment: {probe_environment}
Project instructions and safety policy: {project_instructions}

Decision contract clauses: {relevant_clauses}
Route: {route_record}
Probe card:
{probe_card}

Before running anything, verify that the environment is isolated, the stated
success and failure observations are distinguishable, and all planned
mutations fit the card. If any check fails, stop and return BLOCKED with the
exact mismatch.

Run only the bounded probe. Do not alter the production worktree, shared data,
persistent services, project lockfiles/configuration, or the canonical record.
Do not add scope after seeing the result. Observe existing host approval and
security rules.

Return:
1. ENVIRONMENT: revision, relevant versions, isolation verification.
2. COMMANDS: exact reproducible commands or actions.
3. OBSERVATIONS: raw result needed for the decision; distinguish it from
   interpretation.
4. ORACLE RESULT: success, failure, inconclusive, or blocked.
5. DECISION EFFECT: route status and clause impact dictated by the probe card.
6. LIMITATIONS: what this probe did not establish.
7. EVIDENCE ROWS: candidate E-xx entries.
8. CLEANUP: action and verified result.

If cleanup cannot be verified, flag it prominently. Do not recommend merging
any probe artifact.
```

## Fresh pattern and simplicity challenger

Run this after synthesis on the actual leading route. Use a fresh worker that
did not author or scout the route when possible.

```text
You are the fresh adversarial reviewer for a technical route selection. Work
read-only in {project_root} at revision {revision}. Read
{project_instructions} first.

Decision contract:
{decision_contract}

Pattern baseline:
{pattern_baseline}

Route registry and evidence ledger:
{registry_and_evidence}

Actual leading route and proposed handoff:
{leading_route_and_handoff}

Try to disprove that this route is ready for SELECTED. Inspect the actual
repository references and evidence. Search specifically for:
- a missed established pattern or smaller mechanism;
- a contract clause supported only by assertion or weak proxy evidence;
- unnecessary dependencies, abstractions, interfaces, layers, or concepts;
- an unjustified deviation from repository conventions;
- hidden migration, rollback, compatibility, concurrency, resource, security,
  or operational work;
- a listed non-solution or a concrete counterexample; and
- evidence whose environment does not reproduce the claimed constraint.

Return findings in severity order. For each finding include:
1. FINDING ID and affected route/clause.
2. CLAIM CHALLENGED.
3. EVIDENCE or the label REASONED-ONLY.
4. REPRODUCER / VERIFICATION PATH.
5. EFFECT: reject route, require focused probe, narrow handoff, or no effect.
6. SMALLEST REMEDIATION.

Then return one gate result: PASS, FAIL, or INSUFFICIENT EVIDENCE. PASS means no
material contract, pattern-preservation, or simplicity gate gap remains; it is
not a statement of certainty. Do not propose unrelated improvements or
implement fixes.
```

## Sequential fallback

When no worker facility exists, run the prompts as separate coordinator passes:

1. Freeze the shared inputs in the canonical record.
2. Start a fresh scratch note for one family and read only the contract,
   baseline, and that family assignment.
3. Complete and seal the raw return before reading another family's return.
4. Repeat only for materially different families.
5. Synthesize the sealed returns in the canonical record.
6. Run the challenger prompt as a separate pass focused on falsification.

Do not simulate independence by rewriting the same favored design with
different names. If prior conclusions cannot be kept out of a sequential pass,
label the independence limitation in the evidence ledger.
