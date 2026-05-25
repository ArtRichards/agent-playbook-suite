# Milestone Playbook

Step-by-step procedure the `create-milestones` skill drives.
Short trigger lives in [`../SKILL.md`](../SKILL.md); substance
lives here.

The 10-phase TDD reference lives next door in
[`tdd-phases.md`](tdd-phases.md). Read that before driving any
single phase.

## When this applies

The user wants to create, advance, or complete a milestone for
a project whose foundation work is complete (the Definition of
Ready doc is `Lifecycle: active`).

## Bootstrap (Step 0) — verify the foundation

1. **Locate the docs root.** Walk up from the project location
   for `.docs.toml`. If absent → redirect to
   `project-foundation`. (Do not bootstrap a docs root here.)
2. **Verify the Definition of Ready is active:**
   ```sh
   docs list --root <root> --project <p> --role reference --lifecycle active
   ```
   Output must include `definition-of-ready.md`. If it shows
   `draft` or is missing → redirect to `project-foundation`.
3. **Verify mechanical hygiene.** `docs check <root> --stale 14`
   must exit 0 or 1. Exit 2 = lifecycle drift or broken refs in
   the foundation; fix before adding milestone work.
4. **Read the milestone-plan.**
   `docs list --root <root> --project <p> --role plan --lifecycle active`
   should show `milestone-plan.md`. Read it for M1..Mn.
5. **Read the status doc** for the latest narrative.

## Step 1 — Identify the milestone

From `milestone-plan.md`, pick the next milestone (usually the
lowest-numbered one not yet `active` or `archived`). Slug it
`m<N>` or `m<N>-<short-topic>` (e.g. `m1-fetch-and-parse`).

Confirm the slug with the user before authoring.

## Step 2 — Create the milestone artifacts

Two docs per milestone, paired:

### 2a. Task plan — `Role: milestone`, `Lifecycle: draft`

```sh
docs new milestone <slug> --project <p> --title "<M<N>> — <Title>" --body-from - <<'EOF'
## Overview

- Milestone: <M<N>>
- Title: <Title>
- Surface: <what this milestone delivers>
- Progress: Planned   (prose — Lifecycle: in the metadata block is the controlled vocab)

### Goal

<one paragraph — what success looks like>

### Requirements

- <bulleted functional + non-functional requirements>

### Deliverables

- [ ] Schemas/contracts
- [ ] Implementation
- [ ] Tests (RED → GREEN)
- [ ] Data/fixtures
- [ ] Documentation updates

## Current State Analysis

- Existing code: <what exists>
- Missing: <gaps>
- Known issues: <bugs/debt>

## TDD Implementation Plan

Phases follow the canonical 10-phase TDD methodology. Fill in
per-phase objectives, files, and exit criteria below as the
plan solidifies — each phase section needs `Objective:`,
`Files:`, `Exit:`.

### Phase 1: Define Contract
- Objective:
- Files:
- Exit:

### Phase 2: Write Tests (RED)
- Objective:
- Files:
- Exit:

(...repeat for Phases 3-10 — full names in tdd-phases.md...)

## Phase Checklist

- [ ] Phase 1 — Define Contract
- [ ] Phase 2 — Write Tests (RED)
- [ ] Phase 3 — Create Data/Fixtures
- [ ] Phase 4 — Run Tests (RED Baseline)
- [ ] Phase 5 — Update Base Interfaces
- [ ] Phase 6 — Implement Offline/Core Path
- [ ] Phase 7 — Update Tool/Wrapper Layer
- [ ] Phase 8 — Run Tests (GREEN)
- [ ] Phase 9 — Implement Online/Integration
- [ ] Phase 10 — Quality, Docs, Refactor

## Decisions

<key choices and rationale — append as you go>

## Success Criteria

<what must be true for completion>
EOF
```

### 2b. Implementation log — `Role: log`, `Lifecycle: active`

```sh
docs new log <slug>-impl --project <p> --title "<M<N>> — Implementation Log" --body-from - <<'EOF'
## Overview

Chronological log of work on <M<N>>. Append a section per
phase with objective, files changed, actions, test results,
decisions.

## TDD Phase Progress

| Phase | Progress |
|---|---|
| 1. Define Contract | Pending |
| 2. Write Tests (RED) | Pending |
| 3. Create Data/Fixtures | Pending |
| 4. Run Tests (RED Baseline) | Pending |
| 5. Update Base Interfaces | Pending |
| 6. Implement Offline/Core Path | Pending |
| 7. Update Tool/Wrapper Layer | Pending |
| 8. Run Tests (GREEN) | Pending |
| 9. Implement Online/Integration | Pending |
| 10. Quality, Docs, Refactor | Pending |

## Phase 1 — Define Contract

_Not started._
EOF
```

### 2c. Pair the two docs

Add `Related:` typed edges (careful Edit on the metadata block;
`Related:` is the only metadata field this skill extends after
`docs new`).

To `<slug>.md`:
- `parent-of: <slug>-impl.md`
- `child-of: milestone-plan.md`
- `implements: charter.md`
- any relevant `pairs-with:` (architecture, test-strategy, etc.)

To `<slug>-impl.md`:
- `pairs-with: <slug>.md` (bidirectional reverse of `parent-of`)

### 2d. Flip the milestone doc to active

Edit `Lifecycle: draft` → `Lifecycle: active` in
`<slug>.md`. Then:

```sh
docs touch <slug>.md
docs index <root>
```

Update `status.md`'s "Current milestone" section to point at
the new milestone; `docs touch status.md`.

## Step 3 — Walk the TDD phases

Drive phases 1-10 one at a time. Full per-phase procedure in
[`tdd-phases.md`](tdd-phases.md). The cadence per phase:

1. State phase number, name, objective.
2. Ask clarifying questions if the plan's phase section is
   sparse.
3. Do the work (code, tests).
4. Append a phase section to the impl log via body edit:

   ```markdown
   ## Phase N — <Phase Name> (<YYYY-MM-DD>)

   ### Objective
   <what this phase accomplished>

   ### Files Changed
   | File | Action | Notes |
   |------|--------|-------|

   ### Actions Taken
   - <bulleted list>

   ### Test Results
   - <test counts, pass/fail, notes>

   ### Issues/Decisions
   - <problems, decisions, rationale>
   ```

5. Flip that phase's progress-table cell from `Pending` to
   `Complete`.
6. Tick the matching `[ ]` → `[x]` in the milestone doc's
   Phase Checklist.
7. `docs touch <slug>.md <slug>-impl.md status.md`.
8. Update `status.md`'s "Current Phase" line.
9. `docs index <root>` → `docs check <root> --stale 14`.
   Exit 2 → fix before next phase.
10. Confirm with the user before starting the next phase.

## Step 4 — Complete the milestone

When Phase 10 wraps up:

1. **Append a completion summary** to the impl log:
   verification results (commands + test counts), files
   added/modified/removed, documentation updated, lessons
   learned, pre-existing issues found.
2. **Append a completion summary** to the milestone doc under
   `## Milestone-completion summary`.
3. **Archive with cascade:**
   ```sh
   docs archive <slug>.md --reason "Milestone <M<N>> complete" --cascade
   ```
   `--cascade` walks `Related: pairs-with` and `Related:
   child-of` one hop and prompts for each. Accept for the
   impl log (`<slug>-impl.md`); decline for long-lived
   parents (`milestone-plan.md`, `charter.md`).

   Result: both docs moved to `<root>/archive/<today>/`, both
   `Lifecycle: archived`, INDEX regenerated.
4. **Update `status.md`:**
   - "Current milestone" → next milestone (or "Project
     complete").
   - Append a row to the milestone-progress table with the
     archive date.
   - `docs touch status.md`.
5. **Run the full gate** once more:
   ```sh
   docs check <root> --stale 14
   ```
   Exit 0 or 1. Errors block declaring completion.
6. **Ask the user** about the next milestone.

## Quality gate commands (per phase, adapt to project)

Project-side gate — substitute your stack's commands:

```sh
make format    # or: black . / prettier --write .
make lint      # or: ruff check . / eslint .
make typecheck # or: mypy . / tsc --noEmit
make test      # or: pytest / npm test
```

Plus the docs-side gate, always:

```sh
docs check <root> --stale 14
docs index <root>
```

## Wrapping up a phase or step — sync-and-commit

When a phase, step, or milestone is ready to commit, invoke the
`sync-and-commit` skill. It runs the verification checklist
(consistency / completeness / accuracy), syncs the docs tree
(touches, indexes, checks), and commits with the project's
conventions from CLAUDE.md. Push happens only on feature/
milestone branches with a remote — never on `main`.

`ship-milestone` invokes sync-and-commit automatically at the
end of each step; for interactive use here, invoke it manually
when the phase or milestone is ready.

## Bug fixes and features inside a milestone

- **Feature inside a milestone:** add a section to the
  milestone's task plan; track in the impl log under the
  relevant phase. No new doc.
- **Bug fix:** scoped → impl log under that milestone.
  Cross-cutting → `docs new postmortem <slug>` (incident
  retrospective) or `docs new decision <slug>` (codified fix
  choice).
- **Hot-fix outside the TDD flow:** log to
  `decision-log.md` so the audit trail stays complete.

## Parallel milestones

Supported (rare):

- Each milestone is independent: separate `<slug>.md` +
  `<slug>-impl.md` pair, both `Lifecycle: active`.
- `status.md`'s "Current milestone" becomes a list.
- `docs list --role milestone --project <p> --lifecycle active`
  shows the parallel set.
- Archive each as it completes; cascade is per-archive call.

## Worked example: link-checker M1

Continues the worked example from the `project-foundation`
skill — a small fictional CLI that crawls a website and reports
broken links. Foundation is complete; DoR is `active`; M1 is
"Fetch and parse" (HTTP client, link extraction, in-memory
crawl). Working directory: `~/code/link-checker/docs/specs/`.

### Bootstrap — verify the foundation

```sh
docs list --root . --project link-checker --role reference --lifecycle active
# definition-of-ready.md ... reference ... active   (DoR present)

docs check . --stale 14   # exit 0   (mechanical hygiene clean)

docs list --root . --project link-checker --role plan --lifecycle active
# milestone-plan.md ... plan ... active             (plan present)
```

Reads `milestone-plan.md` — M1 is "Fetch and parse" with no
upstream deps. Confirms slug `m1-fetch-and-parse` with user.

### Create the milestone pair

Uses Step 2a's template. Project-specific deltas filled in:

- Overview Surface: `link_checker.fetch` HTTP client,
  `link_checker.parse` link extractor, `link_checker.crawl`
  in-memory orchestrator.
- Goal: given a seed URL, BFS-crawl same-origin links to
  depth N, return `(url, status_code, depth)` tuples. No
  persistence (M2 owns that).
- Phase 1 sketch: `Objective: pydantic models for Link,
  CrawlResult, CrawlConfig. Files:
  src/link_checker/models.py. Exit: mypy clean, no
  implementation.`
- Success criteria: crawl the 20-page fixture site in < 5s,
  100% link surfacing, dedupe, respect max_depth.

Then the impl log via Step 2b's template (no project-specific
delta beyond title).

Add `Related:` edges per Step 2c. Flip
`Lifecycle: draft → active` in
`m1-fetch-and-parse.md`. `docs touch`. `docs index .`.

Update `status.md`'s "Current milestone" → M1; `docs touch
status.md`.

### Walk Phase 1 (the cadence)

Phase 1 work: create `src/link_checker/models.py` with three
pydantic models. `mypy` clean. Append to impl log:

```markdown
## Phase 1 — Define Contract (2026-05-25)

### Objective
Establish pydantic models for Link, CrawlResult, CrawlConfig.

### Files Changed
| File | Action | Notes |
|------|--------|-------|
| src/link_checker/__init__.py | added | empty package marker |
| src/link_checker/models.py | added | three models |
| pyproject.toml | modified | pydantic ~=2.0 dependency |

### Actions Taken
- Link: (url: HttpUrl, source_url: HttpUrl, depth: int).
- CrawlResult: (links, status_code, error).
- CrawlConfig: defaults max_depth=2, timeout=10.0, retries=3.

### Test Results
N/A — Phase 2 owns tests.

### Issues/Decisions
- Chose pydantic over dataclasses for HttpUrl validation.
```

Flip progress cell `| 1. Define Contract | Pending |` →
`| 1. Define Contract | Complete |`. Tick
`[x] Phase 1 — Define Contract` in the milestone doc.

```sh
docs touch m1-fetch-and-parse.md m1-fetch-and-parse-impl.md status.md
docs index .
docs check . --stale 14   # exit 0
```

Confirm with user → Phase 2.

### Phases 2-10 (abridged)

Same cadence per phase: do the work, append a log section,
flip the progress cell, tick the checklist, `docs touch`,
`docs check`, confirm. Phase 4 captures RED baseline verbatim;
Phase 8 captures GREEN verbatim. Phase 9 has no online surface
for M1 — repurposed as dogfooding against the 20-page fixture
site, with measured metrics (crawl in 3.2s, 100% broken-link
detection) recorded in the Phase 9 log entry. Phase 10 appends
the milestone-completion summary to both docs.

### Milestone completion

```sh
docs archive m1-fetch-and-parse.md \
  --reason "Milestone M1 complete; fixture site crawled in 3.2s, 100% detection" \
  --cascade
```

Cascade prompt: archive `m1-fetch-and-parse-impl.md`
(`pairs-with` target)? Yes. Both docs move to
`archive/2026-06-12/`; both `Lifecycle: archived`; INDEX
regenerated.

Update `status.md`:

- "Current milestone" → M2 (Persistence).
- Milestone progress row:
  `| M1 | Complete (2026-06-12) | [archived](archive/2026-06-12/m1-fetch-and-parse.md) | [archived](archive/2026-06-12/m1-fetch-and-parse-impl.md) |`

```sh
docs touch status.md
docs check . --stale 14   # exit 0
```

Ask user about M2.

### Post-archive

```sh
$ docs list --root . --project link-checker --role milestone
m2-persistence.md                          milestone   draft
archive/2026-06-12/m1-fetch-and-parse.md   milestone   archived

$ ls archive/2026-06-12/
m1-fetch-and-parse-impl.md
m1-fetch-and-parse.md
```

M1's plan and log live together in the dated archive. Active
tree shows M2 as next. Audit trail complete; `docs check`
green.
