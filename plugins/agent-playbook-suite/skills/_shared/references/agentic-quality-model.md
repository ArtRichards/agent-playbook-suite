# Agentic Quality Model

## Purpose

This project uses risk-aware agentic TDD. Visible tests drive implementation,
but visible tests are not sufficient evidence of intent. Each milestone must
define a contract, visible red tests, hidden/generalization strategy, adequacy
checks, technical gates, and product acceptance criteria appropriate to its
risk level.

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

Product tests are default-suite obligations. Non-product checks are explicit
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

## Risk levels

### Lite

Use for internal tools, low-impact refactors, documentation-adjacent
automation, and non-critical admin paths.

Required:
- contract;
- visible tests;
- lint/type/build where configured;
- docs check;
- ordinary review.

### Standard

Use for customer-facing product paths, ordinary APIs, medium-risk refactors,
and behavior used by more than one person/team.

Required:
- Lite gates;
- coverage report if configured;
- hidden/generalization smoke where configured;
- property/stateful smoke where applicable;
- mock audit;
- reviewer signoff on contract/test adequacy.

### High

Use for auth, billing, security, permissions, privacy, data integrity,
migrations, concurrency, incident response, public APIs, and
performance-sensitive core paths.

Required:
- Standard gates;
- operator approval after RED baseline before implementation continues;
- benchmark/security/schema/migration/rollback checks where applicable;
- mutation smoke or mutation baseline where configured;
- fresh-eyes review signoff;
- explicit approval for skipped deep gates.

## Required metrics where available

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

## Hidden-test policy

- Actual hidden/private cases must not be pasted into milestone docs, visible
  tests, implementation prompts, or implementation-agent context.
- It is acceptable to record hidden-test categories, owners, command names,
  and summaries.
- If the repo is fully visible to the implementation agent, assume hidden
  cases are not truly hidden and compensate with mutation, property/stateful,
  fuzz, metamorphic, and review-agent checks.

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
- Do not weaken, skip, or delete tests or required explicit checks unless the
  contract changed and the decision is logged.
- Do not treat green visible tests as sufficient when the risk level requires
  deeper gates.
- Do not allow simplification to reduce test adequacy silently.
