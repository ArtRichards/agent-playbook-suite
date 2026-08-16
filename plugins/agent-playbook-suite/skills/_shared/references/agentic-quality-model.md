# Agentic Quality Model

## Purpose

This project uses risk-aware agentic TDD. Visible tests drive implementation,
but visible tests are not always sufficient evidence of intent. Each milestone
should define a contract, visible red tests or explicit checks, and the smallest
useful set of technical gates and acceptance evidence appropriate to its risk
level.

## Test layers

| Layer | Purpose | Examples | Typical timing |
|---|---|---|---|
| Contract layer | Make intended behavior explicit before coding | acceptance criteria, invariants, error cases, schemas, non-functional constraints | before tests |
| Visible red tests | Drive implementation behavior | unit, component, integration, API, scenario tests visible to implementer | every milestone / PR |
| Hidden/generalization tests | Detect overfitting to visible tests | held-out edge cases, adversarial examples, alternate fixtures, randomized seeds | hidden CI / nightly / release |
| Adequacy tests | Test whether tests are strong enough | mutation, property/stateful, fuzz, metamorphic, prompt-variation checks | nightly / release / high-risk PR |
| Technical gates | Catch structural and non-functional defects | lint, typecheck, build, security, schemas, migrations, benchmark deltas | every PR or risk-gated |
| Product acceptance | Validate user-visible behavior in context | E2E, visual/manual review, CLI dogfooding, benchmark acceptance | PR end / release |

## Product tests vs non-product checks

Product tests validate shipped behavior: runtime code, public APIs, CLI
behavior, integrations, regressions, and externally binding product
contracts. Non-product checks validate project artifacts: preparatory docs,
planning docs, handoff contracts, milestone docs, implementation logs,
metadata, links, and workflow gates. They may use pytest-style assertions, but
they are not product tests by default.

Product tests normally live in the default product suite. Non-product checks
are run only when the milestone or project context names them as explicit
workflow gates.

## Umbrella categories

- Product tests: acceptance criteria, end-to-end journeys, user-visible
  scenarios, business rules expressed in domain language.
- Functional tests: unit, component, integration, API/schema, contract,
  property-based, and stateful tests.
- Technical tests: type checks, linting, build/package validation, security
  scans, migration checks, benchmark/performance, reliability checks.
- Cross-cutting adequacy tests: hidden tests, mutation testing, fuzzing,
  adversarial/metamorphic tests, prompt-variation checks, mock audits.

## Check calibration

Prefer the least constraining check that gives real confidence in the
intended behavior. Check semantic behavior first; do not freeze incidental
representation — byte-exact goldens, exhaustive snapshots, and
change-detector assertions that mirror the implementation overconstrain
later change unless the exact representation is itself the contract.
Overconstrained tests are a defect for review to flag, just like
under-constrained ones: they push complexity into the system instead of
checking correctness.

Name test suites and cases for the scenario and observable behavior in plain
domain language. Make test-runner output understandable without opening a
milestone or decision record. Keep milestone IDs, decision numbers, review
labels, and similar provenance out of test display names. When provenance is
useful, record it in the milestone Decisions section, test matrix, or a nearby
comment or doc, and make every reference resolvable without guessing. Use
whichever readable path, link, or qualified identifier fits the repository.
Apply this rule to new tests and tests actively modified during the work; do
not require a repository-wide rename.

In Kent Beck's test-desiderata terms: good tests are behavioral —
sensitive to changes in the behavior of the code under test — and
structure-insensitive — their result does not change when only the
code's structure changes.

## Risk levels

### Choosing a level

Default to Standard for ordinary product or workflow changes. Reserve Lite
for docs, internal/admin work, and low-blast-radius changes. Propose High
only with an explicit trigger from the High list below: state the reason
and confirm with the operator before recording it — never assign High
unilaterally. Record the agreed level and its reasoning wherever the
milestone or strategy doc asks for it.

### Lite

Use for internal tools, low-impact refactors, documentation-adjacent
automation, and non-critical admin paths.

Default gate set:
- contract;
- visible tests or explicit checks for the changed behavior;
- lint/type/build where configured;
- docs check if the project uses docs-cli;
- ordinary review.

### Standard

Use for customer-facing product paths, ordinary APIs, medium-risk refactors,
and behavior used by more than one person/team.

Default gate set:
- Lite gates;
- coverage report if the project already produces one;
- hidden/generalization smoke where configured;
- property/stateful smoke where the contract is stateful or invariant-heavy;
- mock audit when mocks are added or expanded;
- reviewer signoff on contract/test adequacy when review is available.

### High

Use for auth, billing, security, permissions, privacy, data integrity,
migrations, concurrency, incident response, public APIs, and
performance-sensitive core paths.

Default gate set:
- Standard gates;
- operator approval after RED baseline before implementation continues, unless
  project policy allows automatic continuation;
- benchmark/security/schema/migration/rollback checks where they match the
  risk;
- mutation smoke or mutation baseline where configured;
- fresh-eyes review signoff when available;
- explicit approval or an open entry in the owning project's follow-up log
  (`<project-path>followup-log.md` in a shared docs root) for skipped deep
  gates selected for the milestone.

## Solution uncertainty

Product risk controls verification depth. Solution uncertainty controls whether
the workflow needs technical route exploration. Treat them independently: a
high-risk change can follow a well-established route, while a low-impact novel
problem can have high solution uncertainty.

Do not invoke `explore` because an agent is merely unfamiliar with the code or
lacks confidence. Automatic exploration requires a named technical decision and
downstream consumer, an unresolved result after repository inspection or a cheap
probe, and at least one hard signal:

- an acceptance-critical technical assumption remains unverified;
- no established repository or authoritative upstream pattern fits the core
  mechanism of a genuinely novel problem;
- materially different mechanisms remain plausible and current evidence cannot
  justify a selection; or
- concrete evidence blocked or invalidated the selected route.

When a signal is present, use the `explore` skill before implementation or in
recovery. Its route selection is fail-closed on **pattern-preservation** and a
**simplicity gate**: cite reference modules and reusable project facilities,
state the expected footprint and new concepts, compare the simplest viable
route, and reject unnecessary dependencies, public surface, indirection, or
abstraction. A route may deviate from an existing pattern only when evidence
shows that pattern cannot satisfy the contract; select the smallest deviation
and record the reason. This adds no approval beyond existing project and risk
policy.

Exploration may use isolated, reversible probes with a stated oracle, but it
does not implement production code. Valid outcomes are a selected route, an
**operator-owned product decision** with evidence and tradeoffs, **no viable
route**, or an exact remaining gap. A blocked or rejected route reopens only for
new evidence, a materially different mechanism, a changed constraint, or
removal of its named external blocker.

## Taste model

Correctness gates decide whether the work solves the problem; taste decides
whether a senior reviewer would merge it. The reference point is not a
universal style guide — it is the surrounding codebase and the milestone
contract. Reward alignment with observed project practice, not imposed
conventions.

### Taste anchors (contract-time)

At contract time (Phase 1), record taste anchors in the milestone doc's
Contract:

- the module(s) or area the change should read like;
- in-project libraries expected to be reused;
- established patterns to follow (error handling, config, logging, naming);
- the expected rough diff footprint;
- the dependency policy (default: no new dependencies without a logged
  decision).

Anchors make most of taste objective up front: violating an anchor is a
contract violation and gates like any other contract clause. Mark anchors
`not applicable` for milestones with no meaningful code surface.

### Taste dimensions

Solution quality, judged against the contract and the codebase:

- Minimality — changes are focused on the milestone's Deliverables; no scope
  creep.
- Approach quality — root-cause fix for bugs, sound design for features; not
  a symptom patch.
- Hygiene — no shortcuts, workarounds, hardcoded values, or code smells.
- Fluency — comfortable with the domain, frameworks, tools, and conventions
  in use.
- Craftsmanship — engineering effort a senior reviewer would approve.

Codebase practice alignment, judged against the surrounding code:

- Style consistency — formatting, naming, and structure match the neighbors.
- Pattern adherence — uses the project's established patterns and idioms.
- Library reuse — reuses libraries already in the project rather than
  introducing alternatives or hand-rolled equivalents.
- Abstraction level — the right abstraction level for this part of the
  codebase.
- Documentation fit — comments and docstrings match the project's style and
  density.

### Gating split

Gate only what is objective; force triage of everything else.

- **Gating (blocker-eligible):** taste-anchor violations; scope creep beyond
  the milestone's Deliverables; hygiene violations (shortcuts, workarounds,
  hardcoded values, code smells); library duplication without a logged
  decision.
- **Must-triage (never dropped, never blocking on its own):** findings on the
  subjective dimensions — approach quality, fluency, craftsmanship, style
  consistency, abstraction level, documentation fit. Every such finding ends
  in exactly one of two states: **fixed**, or **waived with a logged reason**
  (in the implementation log or followup log). Work is not finalized while a
  taste finding is untriaged.

Waiver authority: for Lite and Standard milestones the conductor (or, in
interactive runs, the operator at the phase boundary) may waive with a
recorded reason. High-risk waivers require explicit operator approval.

### Compounding

Recurring taste findings and repeated waivers signal a missing project
convention. Promote them into the project's agent context (CLAUDE.md /
AGENTS.md) or into future milestones' taste anchors, so the gating surface
grows one proven rule at a time — never faster than confidence in each rule.

## Demand-driven chains

Chained steps — phases within a milestone, milestones within a plan —
overproduce by default: each step guesses generously at what downstream will
need, tests entrench the guesses, and fresh agents inherit them as apparent
contract. The counter-principle is information liveness: every output a step
produces must trace forward to a consumer.

### The liveness rule

- Every produced output — return value, field, parameter, threaded context,
  doc section — names its consumer: a downstream step, a contract clause, an
  observable behavior, or an explicit audit need.
- A public output that is neither named in the contract nor asserted by a
  visible test is speculative.
- Speculative outputs are permitted only as logged decisions naming the
  intended future consumer, each with a matching entry in the owning project's
  follow-up log (`<project-path>followup-log.md` in a shared docs root; the
  speculative ledger). An unlogged speculative output is a minimality
  finding and routes through taste triage.

### Record the demand, not the implementation

When work reveals a future need outside the current scope, record it — a
line in the milestone plan or the followup log — instead of building surface
for it. A recorded need is cheap, revisable, and re-evaluated when its
milestone is planned; built-ahead surface is test-entrenched and, at every
milestone boundary, laundered into apparent contract that fresh agents
preserve without question.

### Within a milestone

- Phase 5 designs interfaces pull-style: derived backward from the
  milestone's Deliverables and the phase-2 tests, with each intermediate
  output naming its downstream consumer.
- Phase 9 — the first end-to-end run — walks the chain backward from the
  observable outputs and flags anything produced but never consumed.
- Review checks the inverse of completeness: everything demanded is present,
  and everything present is demanded (traceable to a contract clause, a
  visible test, or a ledger entry).
- Simplify removes dead information flow. When a dead output is contractual,
  simplify surfaces a contract-change proposal instead of removing it
  unilaterally.

### Between milestones

- Milestone deliverables name their consumer: the end user, or a specific
  later milestone. Prefer vertical slices whose outputs are consumed
  immediately; a milestone whose deliverables are only surface for later
  milestones is an exception that needs explicit justification.
- At each milestone's planning, check ledger entries naming that milestone
  as consumer: entries the work consumes are closed; entries it does not
  consume are challenged.
- When the milestone plan changes (insertion, re-scope, drop), sweep the
  ledger: entries whose named consumer vanished are dead, and removing that
  surface becomes an explicit task rather than invisible rot.
- At milestone completion, run the boundary sweep: every speculative output
  left behind gets a ledger entry or is removed now, while it is still known
  to be speculation — after archive it is indistinguishable from contract.

## Useful metrics where available

- visible_pass_rate
- hidden_pass_rate
- hidden_generalization_gap = visible_pass_rate - hidden_pass_rate
- changed-lines coverage
- changed-files branch coverage
- mutation score on touched modules
- property/stateful result
- fuzz result
- benchmark delta
- security/schema/migration result
- new mocks introduced
- real-path tests covering mocked boundaries
- patch bloat — diff SLOC vs. the plan's expected footprint; an outsized
  ratio triggers a minimality look, nothing more

## Hidden-test policy

- Actual hidden/private cases must not be pasted into milestone docs, visible
  tests, implementation prompts, or implementation-agent context.
- It is acceptable to record hidden-test categories, owners, command names,
  and summaries.
- If the repo is fully visible to the implementation agent, assume hidden
  cases are not truly hidden and consider mutation, property/stateful, fuzz,
  metamorphic, or review-agent checks based on risk.

## Mock policy

- Do not add or expand mocks without justification.
- Prefer real-path tests for domain behavior.
- Mocks are acceptable for explicit external boundaries, slow/paid services,
  nondeterministic dependencies, and failure injection.
- Every new mock should record what real behavior it substitutes and whether a
  real-path test covers the same contract clause.

## Forbidden shortcuts

- Do not special-case visible examples, fixture names, literals, or test-only
  branches.
- Do not weaken, skip, or delete tests or configured explicit checks unless the
  contract changed and the decision is logged.
- Do not treat green visible tests as sufficient when the risk level requires
  or project policy selects deeper gates.
- Do not allow simplification to reduce test adequacy silently.
