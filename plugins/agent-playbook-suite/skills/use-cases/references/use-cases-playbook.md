# Use-Cases Playbook

The procedure the `use-cases` skill drives. Short trigger lives
in [`../SKILL.md`](../SKILL.md); substance lives here.

## When this runs

- Automatically when `project-foundation` hands off (Definition
  of Ready just flipped `active`).
- On direct invocation, any time the operator wants to explore or
  revisit use cases.
- During feedback work, when a feedback item reveals use-case
  information — feedback discovery gathers how users actually use
  the product, and this doc is where that lands.

## Procedure

Resolve the docs root and the owning project's concrete `<project-path>` before
reading or writing. It is empty for a dedicated project root or a
docs-root-relative directory prefix ending in `/` for a shared root, such as
`specs/payments/`. Use the project-foundation handoff when available; on direct
invocation, match the intended `Project:` across its tracker, status, and
foundation docs, and stop for an explained operator choice if more than one
project fits. Run every displayed docs-cli command from the docs root. Qualify
document operands and `Related:` targets with `<project-path>`; same-directory
Markdown body links remain relative.

### 1. Rebuild context

Read the owning foundation docs: `<project-path>charter.md` (problem,
target user, success metric), `<project-path>scope-and-constraints.md`,
`<project-path>architecture.md`, `<project-path>milestone-plan.md`, and
`<project-path>feedback-log.md` (open feedback often encodes use-case signals).
If `<project-path>use-cases.md`
already exists, this run refines it — read it first.

### 2. Propose candidates

Bring suggestions. Draft three to seven candidate use cases from
the foundation docs and your own sense of the product: who the
actor is, what they are trying to accomplish, how they would
walk through the tool, and what good UX feels like for that
walk. Present them as concrete proposals the operator can
correct, merge, split, or discard. When asking what is missing or
which candidates should be primary, explain which foundation or
feedback evidence produced the candidates and how the choice will
affect milestone test priorities. Include at least one project-grounded
example, and recommend a primary set when the evidence supports one.

Stay open-minded: include use cases the foundation docs do not
mention if the product shape suggests them. The goal is to
understand together how a user could really utilize the software
or tool, with the best UX in mind — not to transcribe the
operator's first answer.

### 3. Refine together

Work one use case at a time. For each candidate the operator
keeps, settle: actor, goal, flow, UX notes (what must feel
effortless, what friction is unacceptable), and what a test must
demonstrate for this use case to count as working. Rank the
keepers — the primary set should stay small enough that "tests
focus on these first" is actionable.

If a refinement needs operator input, ask only the unresolved
choice and follow the shared operator-interaction policy. Do not
turn fields already supported by the foundation docs into a blank
questionnaire.

### 4. Author the doc

```sh
docs new spec <project-path>use-cases --project <p> --title "<Project>: Use Cases" --body-from - <<'EOF'
Primary use cases are primary goals: milestone test matrices map
against them, and tests focus on demonstrating them first.
Maintained by the `use-cases` skill — revisit when feedback work
surfaces new use-case information.

## Primary use cases

### UC1 — <name>

- Actor: <who>
- Goal: <what they accomplish>
- Flow: <how they walk through the tool>
- UX notes: <what must feel effortless; unacceptable friction>
- Test focus: <what a test must demonstrate>

## Secondary / deferred use cases

<kept visible, not driving tests yet>

## Anti-use-cases

<what this product deliberately does not serve>

## Test-matrix mapping

| Use case | Milestone(s) | Test-matrix rows |
|---|---|---|
|  |  |  |
EOF
```

Add `Related: pairs-with: <project-path>charter.md` and
`Related: pairs-with: <project-path>test-strategy.md` (careful Edit on the
metadata block, then `docs touch <project-path>use-cases.md`).

Flip `Lifecycle: draft` to `active` once the operator confirms
the primary set; `docs touch <project-path>use-cases.md`; `docs check .`
clean.

### 5. Map to test matrices

Each primary use case names the milestones that deliver it and
the test-matrix rows that demonstrate it. The mapping fills in as
milestones are created — `create-milestones` reads this doc when
drafting a matrix and records the mapping on both sides. Tests
focus primarily on the use cases; checks that pin incidental
representation come second, if at all.

### 6. Maintain

- Note the exploration outcome in `<project-path>foundation-log.md` when run as
  the foundation handoff.
- When feedback work surfaces use-case information, update the
  affected entries and the mapping; `docs touch
  <project-path>use-cases.md`.
- When a use case is promoted, demoted, or retired, record the
  reasoning in `<project-path>decision-log.md`.

## Stop conditions

The exploration is done when the operator confirms the primary
set and each primary use case has an actor, goal, flow, UX notes,
and test focus. If the operator declines the stage, note that in
`<project-path>foundation-log.md` and stop — the stage is optional, and
milestone work may proceed without it.
