# Foundation Playbook

The step-by-step procedure the `project-foundation` skill drives.
Short trigger lives in [`../SKILL.md`](../SKILL.md); substance
lives here so SKILL.md can stay small.

Pair this with [`role-mapping.md`](role-mapping.md) for the
artifact-to-role-and-lifecycle table — every `docs new` call
below uses the role and starting `Lifecycle:` from that table.
Before Phase 4, read the shared
[`milestone-tracker.md`](../../_shared/references/milestone-tracker.md)
contract. It is the sole authority for the tracker schema, semantic
milestone identity, stored execution states, and recognized milestone
relationships.

## When this applies

The user wants to start a new project (or a sub-project) and
needs the front-half foundation work — charter through
Definition of Ready — before any implementation.

## Bootstrap (Step 0)

Before any phase question:

1. **Detect or pick the docs root.** Walk up three directory
   levels from the project location looking for `.docs.toml`.
   - **Found existing root** → inspect it before reusing: is it
     internal project documentation, and does its layout fit this
     project? When it already hosts other projects, reuse it as a
     shared root and place this foundation at a root-relative path
     such as `specs/<project-slug>/`. Otherwise prefer a dedicated
     root whose directory is this project, or ask whether to
     bootstrap one.
   - **No root found** → inspect the existing directories before
     suggesting a location. Classify any candidate documentation
     directory as internal vs public/customer-facing before
     reusing it: public signals include a documentation builder,
     a tool that builds docs for customers, or generated customer
     docs. Never place internal specs inside a public
     documentation surface, or in site/publishing directories
     such as `site/`, `website/`, `pages/`, or `public/`, even
     when they contain Markdown.
     - `docs/` exists and is internal → prefer a dedicated root at
       `<repo-root>/docs/specs/<project-slug>/` when that fits the
       layout. Single-project repos may use `docs/specs/` directly.
     - `docs/` exists but is public/customer-facing → prefer
       `internal-docs/specs/<project-slug>/`, and ask the
       operator before creating it.
     - No usable internal docs directory → suggest
       `<repo-root>/docs/specs/` if a `docs/` directory exists,
       else `<repo-root>/specs/`, and confirm.
   - **Confirmation scales with repository maturity.** In a large
     existing codebase, confirm before creating a new docs root
     or internal specs directory. In a relatively greenfield
     repo, proceed with the preferred default when the layout is
     clear.
   - **Resolve one `<project-path>`.** This is the docs-root-relative
     directory prefix for every live doc in this project, including a
     trailing slash when nonempty. It is empty for a dedicated project
     root and, for example, `specs/payments/` in an existing shared root.
     `Project:` remains the metadata owner; `<project-path>` is what
     disambiguates artifact paths. Do not nest a lone project under a
     newly created parent root merely to add a redundant prefix.

2. **Bootstrap only a new dedicated root** by:
   - Creating the directory.
   - Copying [`docs-toml-template.toml`](docs-toml-template.toml)
     to `<root>/.docs.toml` and substituting the project slug
     into `[project] name`.
   - Changing the working directory to `<root>` and running
     `docs check .` once (empty tree always passes).

   When joining an existing shared root, retain its `.docs.toml`, create only
   the `<project-path>` directory, change the working directory to the found
   root, and validate the existing tree before authoring.

3. **Resolve the project slug** — kebab-case, lowercase, and unique across the
   repository — and bind it to the chosen `<project-path>`. Inspect all
   docs-managed trees and their `Project:` metadata before accepting it; if the
   same slug belongs to another project path, choose a distinct slug rather
   than creating an ambiguous branch namespace. Run every subsequent docs-cli
   command from the docs root. `--root` selects a tree but does not rebase
   relative `FILE`, `SOURCE`, or `TARGET` operands, so it is not a substitute
   for changing directory.
   Every document path operand and every `Related:` target is root-relative and
   begins with `<project-path>`;
   `docs new` receives `<project-path><slug>` without `.md`. Same-directory
   Markdown body links stay relative and omit the prefix.

   For example, a `payments` project in a shared root can use
   `<project-path> = specs/payments/`: create its tracker with
   `docs new plan specs/payments/milestone-plan`, touch it with
   `docs touch specs/payments/milestone-plan.md`, and target its artifacts as
   `specs/payments/payments-<slug>.md` in `Related:` and `docs relate`
   operations. The project-qualified physical stem remains distinct after
   docs-cli flattens it into the archive. A link from its status body to its
   tracker remains
   `[Milestone Plan](milestone-plan.md)`.

   Also derive `<project-branch-prefix>` for generated agent context. Use
   `<project-slug>/` whenever the repository has more than one suite project;
   use the empty string only when one project-wide milestone namespace is
   guaranteed. Freeze that choice once a milestone is activated and reject an
   existing branch root owned by another project instead of renaming it.

4. **Create the five living docs** before any phase question at
   `<project-path>{status,foundation-log,risks,followup-log,feedback-log}.md`.
   `status.md` uses `Role: status`; the others use `Role: log`; all five use
   `Lifecycle: active`. These
   accumulate throughout the project — see the
   [worked example](#worked-example-link-checker) for their
   initial bodies.

   The two project logs are the single home for open items, kept
   outside milestone docs so nothing is lost when milestones
   archive: `followup-log.md` holds engineering follow-ups
   (adequacy gaps, skipped deep gates, deferred test work) in a
   tight entry shape; `feedback-log.md` holds operator feedback,
   ideas, and scope thoughts as dated intake entries. Each log
   embeds its own entry template in its body — skills appending
   entries follow the template found in the file. Milestone docs
   reference open entries rather than holding them inline; when a
   milestone incorporates an item, its content moves into that
   milestone's docs and the log entry is removed.

## Ground every phase in investigation

Foundation questions work best when the wizard arrives informed.
Before asking a phase's questions, inspect what already exists —
code, build files, existing tests, READMEs, deployment configs —
and fold the findings into the questions: propose answers the
evidence supports instead of presenting blank questionnaires, and
let the operator correct a concrete proposal rather than fill in
a form.

Treat each phase's `Ask` list as discovery coverage, not a script.
Ask only what remains unresolved after inspection. For every
necessary question, follow the shared operator-interaction policy:
explain where it came from, why it matters at this phase, and what
the answer changes; include a concise project-grounded example or a
clearly labeled hypothetical; recommend the evidence-supported answer
or state that there is no strong recommendation. Keep the prose
proportional to the actual decision rather than forcing a fixed format.

If the operator asks for it, launch a thorough investigation of
the entire product's foundation — architecture, module
boundaries, dependencies, data flows, test coverage, operational
surfaces — before or during foundation work. Record the findings
in `<project-path>foundation-log.md` (and
`<project-path>architecture.md` once it exists) so
the investigation outlives the session.

## Authoring docs without harness friction

Every `docs new` call in this playbook uses the `--body-from`
flag so metadata block and body land in a single Bash call, with
no Read-before-Write round trip:

```sh
docs new <role> <project-path><slug> --project <p> --title "<H1>" --body-from - <<'EOF'
## <First section heading>

<body content>
EOF
```

`docs new` writes the frontmatter (`Lifecycle: draft`,
`Role: <role>`, `Project: <p>`, `Updated: <today>`) under the
synthesised H1, then appends the body verbatim.

**Never** include a metadata block in piped body content.
`docs new --body-from` refuses content whose first 20 lines
contain a `Label: value` line and exits 2 — agent must
self-correct.

## Adding `Related:` edges

For free-form verbs such as `pairs-with`, `implements`, and `references`,
use a careful Edit on the metadata block to add
`- <verb>: <project-path><target>.md` under `Related:` (or insert the block if
it was not scaffolded), then run `docs touch <project-path><file>.md`.

For the recognized reciprocal pairs `precedes`/`follows`,
`depends-on`/`required-by`, and `blocks`/`blocked-by`, never hand-author
either half. Use
`docs relate add <project-path><source>.md <verb> <project-path><target>.md`
or the corresponding `docs relate remove` so docs-cli validates and
updates both endpoints together. If either endpoint is archived, include
the required one-line `--reason`. Foundation normally has no materialized
milestone endpoints yet; synchronize these relationships later when a
milestone artifact exists, as directed by the shared tracker contract.

## Phase 0 — Intake & Alignment

Ask:

- What problem are you trying to solve?
- Who is the target user?
- What is the measurable success metric?
- What are the non-goals?

Author:

```sh
docs new charter <project-path>charter --project <p> --title "<Project>: Charter" --body-from - <<'EOF'
## What we're building

<problem statement, target user, measurable success metric>

## Why

<motivation; what hurts today>

## Non-goals

<explicit out-of-scope items>
EOF
```

Lifecycle starts `draft`; flips to `active` once DoR is green.

## Phase 1 — Scope & Constraints

Ask:

- What is in scope?
- What is explicitly out of scope?
- What assumptions are you making?
- What are hard constraints (time, budget, tech stack, compliance)?
- What open questions need answers before proceeding?

Author `docs new spec <project-path>scope-and-constraints` with body sections:
**In scope**, **Out of scope**, **Assumptions**, **Constraints**,
**Open questions**. Add
`Related: pairs-with: <project-path>charter.md`.

## Phase 2 — Stakeholders & Interfaces

Ask:

- Who are the stakeholders?
- What external systems/APIs will this integrate with?
- What data will be consumed or produced?
- Who owns each interface?

Author `docs new reference <project-path>stakeholders` with body sections:
**Stakeholders**, **Interfaces (consumed)**, **Interfaces
(produced)**, **Data flows**, **Ownership**. Add
`Related: pairs-with: <project-path>charter.md`.

If solo with no external interfaces, write a single paragraph
acknowledging that — the doc still exists as a DoR checkbox.
Do not skip.

## Phase 3 — Solution Options & Architecture

Ask:

- What are 2-3 possible approaches?
- For each option, what are the pros and cons?
- Which option do you prefer and why?
- What are the main modules/components?
- How does data flow between them?

Before comparing routes or invoking optional exploration, create the canonical
decision record:

1. `docs new decision <project-path>options-comparison` — keep
   `Lifecycle: draft` while the choice is open. Start the body with **Context**,
   **Options considered** (one
   H3 per option, with pros/cons), **Selected approach**, and **Rationale**.

Use the ordinary comparison for established solution shapes. Invoke `explore`
automatically only when the shared quality model's solution-uncertainty gates
are met: the architecture decision and its consumer are named, direct
inspection or a cheap probe did not resolve it, and the problem is genuinely
novel or an acceptance-critical route remains clearly uncertain. Do not invoke
it for ordinary unfamiliarity. Keep the approach registry and evidence in the
existing draft `<project-path>options-comparison.md`; do not create a parallel
exploration doc. A selected route feeds the architecture sketch, an
operator-owned product
decision is surfaced with its tradeoffs, and an unresolved acceptance-critical
gap keeps the relevant Definition of Ready item unready. When an ordinary
comparison settles the choice, or `explore` returns `SELECTED` (including after
an operator answer is bound), finish the decision record's **Selected
approach** and **Rationale** sections and make it active. `OPERATOR DECISION`,
`NO VIABLE ROUTE`, and `INSUFFICIENT EVIDENCE` leave it draft with the exact
gap and next owner/action recorded.

Author the remaining two docs:

2. `docs new sketch <project-path>architecture` — `Lifecycle: draft`. Body:
   **Shape**, **Data flow**, **Integration points**. Add
   `Related: implements: <project-path>charter.md`, `Related: pairs-with:
   <project-path>options-comparison.md`. Graduates to `Role: reference` when
   authoritative. If no route is selected, record only known constraints and
   gaps and keep the sketch draft.
3. `docs new log <project-path>decision-log` — `Lifecycle: active`. Ongoing
   log of choices made during the project; the first entry summarises the
   Phase 3 disposition and, once settled, the architecture choice. Append dated
   `## YYYY-MM-DD — <one-line>` entries as decisions accumulate.

## Phase 4 — Delivery Strategy & Milestones

Ask:

- What are the major milestones (3-5 checkpoints)?
- What does each deliver?
- What meaningful semantic slug identifies each one?
- Which milestones are durable prerequisites, and which may proceed in
  parallel?
- Where are the demo/review checkpoints?

Author `docs new plan <project-path>milestone-plan` with exactly one canonical
table in this shape and column order:

```markdown
| Order | Milestone | State | Depends on | Notes |
|---:|---|---|---|---|
| 100 | fetch-and-parse | planned | — | Crawl and extraction contract |
| 200 | persistence | planned | fetch-and-parse | Resume prior crawls |
| 200 | reporting | planned | fetch-and-parse | Human and machine output |
```

Use project-unique, lowercase kebab-case semantic slugs. Start distinct
cohorts at `100`, `200`, `300`, and so on; equal `Order` values mean the
rows may proceed in parallel. Every foundation row starts `planned`.
`Depends on` is `—` or a comma-separated list of tracker slugs. Because
foundation does not create milestone artifacts, keep the `Milestone` cells
plain text rather than creating stubs merely to add links. Later
materialization replaces the plain cell with a link whose text remains the
semantic slug.

Precompute and validate each row's future physical `<artifact-stem>` without
storing it as a tracker column: use `<slug>` in a dedicated project root and a
project-qualified basename such as `<project>-<slug>` in a shared root.
Docs-cli flattens basenames into `archive/<date>/`, so a directory prefix alone
cannot protect same-slug projects during archival. Reject any proposed stem
whose milestone, `-impl`, or `-test-matrix` basename collides root-globally.
When materialized, the row link target is the same-directory relative
`<artifact-stem>.md`; `Depends on` and link text continue to use semantic
slugs. Apply the shared contract's reserved-suffix checks too.

The plan may follow the table with **Sequencing**, **Milestone details**
(one H3 per semantic slug, with goal, demo checkpoint, and consumer), and
**Buffer notes**. These sections describe scope; the table alone owns
identity, order, stored execution state, dependencies, and derived next
work. `<project-path>status.md` remains a narrative summary and link surface,
not a second scheduler.

Express goals and demo/acceptance criteria as observable outcomes. Clarify
consequential ambiguity; note the source of an expectation only when it is
non-obvious or disputed.

Decompose demand-driven (see the shared quality model's
Demand-driven chains): each milestone's deliverables name their
consumer — the end user, or a specific later milestone. Prefer
vertical slices whose outputs are consumed immediately over
horizontal layers ("models-only", followed by "services-only");
a milestone that delivers only surface for later milestones is
an exception that needs explicit justification in the plan.

Add `Related: implements: <project-path>charter.md`,
`Related: pairs-with: <project-path>architecture.md`. Per-milestone task plans
and companions at `<project-path><artifact-stem>.md`,
`<project-path><artifact-stem>-impl.md`, and
`<project-path><artifact-stem>-test-matrix.md` belong to `create-milestones`,
not here. Once those artifacts exist, sequence and dependency relationships
are synchronized with qualified `docs relate` operands under the shared
tracker contract; tracker order never becomes part of semantic identity or
physical naming.

After `<project-path>milestone-plan.md` exists, update
`<project-path>status.md`: preserve its narrative, add the same-directory body
link `[Milestone Plan](milestone-plan.md)`, and remove copied tracker rows or
independent schedule/next claims. Then run, in order:

```sh
docs touch <project-path>milestone-plan.md <project-path>status.md
docs index .
docs check . --stale 14
```

## Phase 5 — Environment & Tooling

Ask:

- What runtime/language/framework will be used?
- What are the build commands?
- What are the test commands?
- What access/credentials are needed?
- Any blockers to getting started?

Author `docs new runbook <project-path>env-and-tooling` with body sections:
**Runtimes**, **Build commands**, **Test commands**,
**Access/credentials**, **Blockers**. `runbook` because the
content describes commands to run, not concepts to think about.

## Phase 6 — Data & Compliance

Ask:

- What data sources will be used?
- Synthetic or real data for development/testing?
- Data quality expectations?
- Privacy/security/compliance constraints?

Author `docs new plan <project-path>data-plan` with body sections:
**Data sources**, **Synthetic vs. real approach**, **Quality
expectations**, **Privacy/security/compliance**.

## Phase 7 — Risk-Aware Validation Strategy

Ask:

- What product, functional, technical, and cross-cutting adequacy
  tests will you write?
- Which preparatory, planning, handoff, or workflow artifacts need
  explicit non-product checks?
- Which project areas or milestone families are Lite, Standard, or
  High risk, and why?
- What commands make up the fast PR gate?
- What commands make up the deep/nightly/release gate, or are they
  explicitly not configured yet?
- Where will hidden/generalization checks live, who owns them, and
  what must not be exposed to implementation agents?
- What mocks are acceptable, which require justification, and where
  is real-path coverage required?
- What human/operator approval triggers should stop implementation?

**Propose risk levels, never assign them.** Investigate the repo
or product first, then propose a level per area with plain-language
reasoning proportional to the risk and confirm with the operator.
Default to Standard for ordinary product or workflow changes;
reserve Lite for docs,
internal/admin work, and low-blast-radius changes. Propose High
only with an explicit reason from the shared quality model's High
triggers (auth, billing, security, privacy, data integrity,
migrations, concurrency, incident response, public APIs,
performance-sensitive core paths) — and get the operator's
explicit approval before recording High. Record the agreed level
and its reasoning in the risk table.

Author `docs new outline <project-path>test-strategy` with body sections:
**Validation taxonomy**, **Risk levels**, **Fast PR gate**,
**Deep/nightly/release gate**, **Hidden-test policy**,
**Mock policy**, **Human approval triggers**, and **Coverage /
adequacy metrics**. `outline` (not `spec`) because the detail fills
in as implementation proceeds; graduates to `spec` later if it
becomes the canonical test contract.

Reuse existing commands for the selected gates. Template entries do not select
extra techniques. If a greenfield harness does not yet exist, record planned
commands and schedule its implementation in a milestone.

Use this starter shape:

```markdown
## Validation taxonomy

- Product tests:
- Non-product checks:
- Functional tests:
- Technical tests:
- Cross-cutting adequacy tests:

Contract-to-test mapping uses the existing `create-milestones` test-matrix
template in `references/milestone-playbook.md`; each milestone fills it in.

## Risk levels

| Area | Risk level | Reason | Selected gates |
|---|---|---|---|
|  | Lite / Standard / High |  |  |

## Fast PR gate

- Format:
- Lint:
- Typecheck:
- Build:
- Visible tests:
- Coverage:
- Property/stateful smoke:
- Security/schema smoke:
- Docs check:

## Deep/nightly/release gate

- Hidden/generalization:
- Mutation:
- Full property/stateful:
- Fuzz:
- Benchmarks:
- Security/dependency scan:
- Migration/rollback rehearsal:

## Hidden-test policy

- Where hidden tests live:
- Who owns them:
- What the implementation agent may see:
- What must never be pasted into implementation prompts:

## Mock policy

- Acceptable mocks:
- Mocks requiring justification:
- Required real-path coverage:

## Human approval triggers

- High-risk areas:
- Unresolved ambiguities:
- Public API/schema changes:
- Auth/security/privacy/billing/data integrity:
- Performance budget changes:
- New or expanded mocks:
```

## Phase 8 — Documentation Plan

Ask:

- What documentation will be created?
- Where will each doc live?
- How often will docs be updated?

Author `docs new plan <project-path>documentation-plan` with body sections:
**Required docs** (table: name + role + owner + cadence),
**Locations**, **Update cadence**.

Most of this is auto-handled by the docs tree itself — the
foundation artifacts are already documented, indexed, and
lifecycle-tracked. The plan only needs to cover docs outside
the docs tree (code-side READMEs, API references, operator
runbooks, **and agent context files** — see below).

### Agent context files: CLAUDE.md and AGENTS.md

`CLAUDE.md` and `AGENTS.md` are project-root files (not docs-tree
artifacts) that give agent sessions enough context to rebuild
understanding from scratch. The `ship-milestone` skill's sub-agents
explicitly read project context as their first action, so this is
load-bearing for autonomous milestone work across Claude Code, Codex,
and compatible hosts.

Run this **after** authoring `<project-path>documentation-plan.md`:

1. **Check for existing host context files** at the repo root:
   ```sh
   ls <repo-root>/CLAUDE.md <repo-root>/AGENTS.md 2>/dev/null
   ```

2. **If both are absent — scaffold at least one host context file.**
   Read [`claude-md-template.md`](claude-md-template.md), fill the
   derived slots from the artifacts created in Phases 0-7, and write
   the rendered file directly to `<repo-root>/CLAUDE.md` and/or
   `<repo-root>/AGENTS.md` depending on the operator's agent host.
   Use the same substantive sections for both files. Confirm with the
   user before writing if any slot is ambiguous (e.g. multiple
   plausible docs-root paths).

3. **If either exists — propose additions, never overwrite.** Read
   each existing context file. Use the detection heuristics in
   [`claude-md-template.md`](claude-md-template.md) to identify which
   fixed sections are missing: Documentation tree, Skill ecosystem,
   TDD methodology, Quality gates and test adequacy, Branch
   conventions, Operator questions and decisions.
   - **All fixed sections present** -> the file is good as-is. Note
     that in `<project-path>foundation-log.md`; do not write an additions file.
   - **One or more missing** -> write `CLAUDE-additions.md` and/or
     `AGENTS-additions.md` next to the existing file with the proposed
     additions (rendered, with derived slots filled). Show the user the
     path and a short summary of what's proposed; let them review and
     merge by hand.

4. **Add context files to `<project-path>documentation-plan.md`'s Required docs
   table** — name `CLAUDE.md` and/or `AGENTS.md`, role "project
   context for coding agents", owner the project owner, cadence
   "updated when skill ecosystem, quality gates, or tooling changes."

5. **Note the action in `<project-path>foundation-log.md`** — one of:
   "scaffolded CLAUDE.md/AGENTS.md from template", "wrote
   context additions for operator review", or "agent context already
   covers the skill ecosystem and quality gates (no changes needed)."

These context files are not in the docs tree, so they have no
`Lifecycle:` metadata block, do not appear in `INDEX.md`, and are not
validated by `docs check`. They are plain prose, hand-edited
thereafter.

## Phase 9 — Definition of Ready (the gate)

Definition of Ready is `Role: reference`. Author:

```sh
docs new reference <project-path>definition-of-ready --project <p> --title "<Project>: Definition of Ready" --body-from - <<'EOF'
Gate-check before implementation begins. Implementation does not start until every item is green.

## Foundation checklist

- [ ] **Charter** states problem, user, success metric, non-goals. → [charter.md](charter.md)
- [ ] **Scope, assumptions, constraints captured.** → [scope-and-constraints.md](scope-and-constraints.md)
- [ ] **Stakeholders/interfaces noted with owners.** → [stakeholders.md](stakeholders.md)
- [ ] **Architecture option chosen; decision recorded.** → [options-comparison.md](options-comparison.md), [architecture.md](architecture.md)
- [ ] **Canonical milestone tracker has semantic slugs, explicit order, allowed states, and valid dependencies.** → [milestone-plan.md](milestone-plan.md)
- [ ] **Environment/tooling validated; access unblocked.** → [env-and-tooling.md](env-and-tooling.md)
- [ ] **Data plan set; compliance/privacy constraints clear.** → [data-plan.md](data-plan.md)
- [ ] **Test strategy outline covers critical paths and fixtures.** → [test-strategy.md](test-strategy.md)
- [ ] **Risk level is assigned for each milestone or project area.** → [test-strategy.md](test-strategy.md)
- [ ] **Fast PR gate commands are documented.** → [test-strategy.md](test-strategy.md)
- [ ] **Deep/nightly/release gate commands are documented or explicitly marked not applicable.** → [test-strategy.md](test-strategy.md)
- [ ] **Hidden/generalization strategy is documented without exposing private cases.** → [test-strategy.md](test-strategy.md)
- [ ] **Test strategy references the existing `create-milestones` contract-to-test matrix template.** → [test-strategy.md](test-strategy.md)
- [ ] **Mock policy exists.** → [test-strategy.md](test-strategy.md)
- [ ] **Human approval triggers are listed.** → [test-strategy.md](test-strategy.md), [risks.md](risks.md)
- [ ] **Codex/Claude agent context is generated or proposed.** → [documentation-plan.md](documentation-plan.md)
- [ ] **Documentation plan set.** → [documentation-plan.md](documentation-plan.md)
- [ ] **Risks logged with owners; go/no-go recorded.** → [risks.md](risks.md)

## Residual risks

<list any risks the gate accepts as known-and-mitigated>

## Decision

<go / no-go / blocked-on-X — and the date>
EOF
```

Add `Related: pairs-with: <project-path>charter.md`,
`Related: pairs-with: <project-path>milestone-plan.md`, and
`Related: pairs-with: <project-path>status.md`.

**Mechanical gate.** Before declaring green:

```sh
docs check . --stale 14
```

Exit-code verdict:

- `0` → mechanically clean. Apply qualitative review (every
  checkbox justified, every link resolves, success metric
  measurable, residual risks logged). Flip the DoR doc and
  every front-half doc from `Lifecycle: draft` to
  `Lifecycle: active`, then perform the qualified status/touch/index/check
  completion sequence below.
- `1` → warnings (medium-confidence inferences or stale docs).
  Review; touch the qualified paths if still correct, or update content, then
  re-run.
- `2` → errors (missing fields, broken refs, lifecycle/location
  drift). Fix before proceeding. DoR cannot pass.

## Completion and hand-off

When DoR flips to `active`:

1. Update `<project-path>status.md` with a narrative foundation-complete
   summary and `[Milestone Plan](milestone-plan.md)`. Remove copied tracker
   rows and independent schedule/next claims.
2. After the lifecycle and status body edits, atomically touch every changed
   qualified path. The batch must include both
   `<project-path>milestone-plan.md` and `<project-path>status.md`.
3. Run `docs index .` and `docs check . --stale 14`; require exit 0.
4. `docs list --root . --lifecycle active --project <p>` shows every
   artifact in the expected state.
5. Hand off the resolved docs root, `<project-path>`, and `Project:` value to
   the `use-cases` skill — it runs automatically after foundation completes
   (optional, but strongly preferred)
   to explore the primary use cases that will guide testing.
6. Then hand off to `create-milestones` for `next milestone` or an
   explicitly named eligible semantic slug.

## Worked example: link-checker

A small fictional project — a CLI that crawls a website and
reports broken links. Project slug: `link-checker`. User
invoked the skill from inside `~/code/link-checker/` (a repo
with a `docs/` directory).

### Step 0 — Bootstrap

Wizard walks up three levels, finds no `.docs.toml`. Default
suggestion (`docs/` exists → `docs/specs/`) confirmed. This is a dedicated
root, so `<project-path>` is empty and the expanded docs-cli operands below
are the bare filenames. Then:

```sh
mkdir -p ~/code/link-checker/docs/specs
cp ~/.claude/skills/project-foundation/references/docs-toml-template.toml \
   ~/code/link-checker/docs/specs/.docs.toml
sed -i 's/<PROJECT_SLUG>/link-checker/' ~/code/link-checker/docs/specs/.docs.toml
cd ~/code/link-checker/docs/specs
docs check .   # exit 0 — empty tree
```

Five living docs created before any phase question:

```sh
docs new status status --project link-checker \
  --title "link-checker: Status" --body-from - <<'EOF'
Narrative project summary. The canonical milestone tracker is created in
Phase 4; once it exists, this doc links to it without copying its rows.

## Project summary

Foundation work is in progress.

## Planning links

_Milestone plan pending Phase 4._
EOF

docs new log foundation-log --project link-checker \
  --title "link-checker: Foundation Log" --body-from - <<'EOF'
Rolling log of decisions and notes during foundation work.

## 2026-05-24 — Bootstrap

Docs root created at `docs/specs/`. Wizard beginning Phase 0.
EOF

docs new log risks --project link-checker \
  --title "link-checker: Risk Register" --body-from - <<'EOF'
Risks identified during and after foundation work. Append a
dated H2 entry per risk with body sub-sections:
**Trigger**, **Mitigation**, **Owner**.

## (no risks logged yet)
EOF

docs new log followup-log --project link-checker \
  --title "link-checker: Follow-up Log" --body-from - <<'EOF'
Open engineering follow-ups: adequacy gaps, skipped deep gates,
deferred test work, and similar items not yet scheduled into a
milestone. This log is the single home for open items — milestone
docs reference an entry while it is open and drop the reference
once it is incorporated. When a milestone takes an item on, move
its content into that milestone's docs and remove the entry here.

Append a dated H2 entry per item with body sub-sections:
**Item**, **Source milestone**, **Risk**, **Owner**, **Status**.

## (no open follow-ups yet)
EOF

docs new log feedback-log --project link-checker \
  --title "link-checker: Feedback Log" --body-from - <<'EOF'
Operator feedback, ideas, and scope thoughts that surface during
work. This log is the single home for open items — when the work
that addresses an entry lands, move its content to where it was
acted on (a milestone doc, the scope spec, a decision entry) and
remove the entry here.

Append a dated H2 entry per item with body sub-sections:
**Source**, **Feedback** (verbatim), **Evidence or example**,
**Use cases affected**, **Planned change**, **Status**,
**Follow-up**.

## (no feedback logged yet)
EOF
```

### Phases 0-8

Wizard asks each phase's questions and authors the artifact
with `docs new --body-from -`, appending a one-line note to
the empty-prefix `foundation-log.md` after each. After all eight phases, the
tree looks like:

```
~/code/link-checker/docs/specs/
├── .docs.toml
├── INDEX.md                     (regenerated by docs index)
├── charter.md                   Role: charter,    Lifecycle: draft
├── scope-and-constraints.md     Role: spec,       Lifecycle: draft
├── stakeholders.md              Role: reference,  Lifecycle: draft
├── options-comparison.md        Role: decision,   Lifecycle: draft
├── architecture.md              Role: sketch,     Lifecycle: draft
├── decision-log.md              Role: log,        Lifecycle: active
├── milestone-plan.md            Role: plan,       Lifecycle: draft
├── env-and-tooling.md           Role: runbook,    Lifecycle: draft
├── data-plan.md                 Role: plan,       Lifecycle: draft
├── test-strategy.md             Role: outline,    Lifecycle: draft
├── documentation-plan.md        Role: plan,       Lifecycle: draft
├── foundation-log.md            Role: log,        Lifecycle: active
├── risks.md                     Role: log,        Lifecycle: active
├── followup-log.md              Role: log,        Lifecycle: active
├── feedback-log.md              Role: log,        Lifecycle: active
└── status.md                    Role: status,     Lifecycle: active
```

`milestone-plan.md` begins with its canonical tracker:

```markdown
| Order | Milestone | State | Depends on | Notes |
|---:|---|---|---|---|
| 100 | fetch-and-parse | planned | — | HTTP client, link extraction, in-memory crawl |
| 200 | persistence | planned | fetch-and-parse | SQLite cache and resume |
| 200 | reporting | planned | fetch-and-parse | HTML/JSON output and summary stats |
```

The semantic slugs remain stable if the operator later changes the two
`200` rows to different cohorts or inserts a new order between them.
Because this example uses a dedicated docs root, each future artifact stem is
the same as its semantic slug; a materialized row would link, for example,
`[fetch-and-parse](fetch-and-parse.md)`. In a shared root, the visible text
would stay `fetch-and-parse` while the relative target would be
`link-checker-fetch-and-parse.md`.
Once the plan exists, the wizard replaces the pending note in `status.md`
with `[Milestone Plan](milestone-plan.md)`, removes any independent schedule
claim, then runs `docs touch milestone-plan.md status.md`, `docs index .`, and
`docs check . --stale 14` in that order.

### Phase 8 addendum — CLAUDE.md scaffold

`~/code/link-checker/CLAUDE.md` doesn't exist (greenfield repo).
Wizard renders `claude-md-template.md` with the derived slots:

- `<project summary>`: first paragraph of `charter.md`.
- `<docs-root-path>`: `docs/specs`.
- `<project-path>`: empty (the docs root is dedicated).
- `<project-slug>`: `link-checker`.
- `<project-branch-prefix>`: empty (the repository has one project-wide
  milestone namespace and `link-checker` is repository-unique).
- Build/test commands: from `env-and-tooling.md` (e.g. `pytest`,
  `ruff check .`, `mypy`).
- Commit conventions: defaulted (no prior git history yet).

Writes to `~/code/link-checker/CLAUDE.md`; notes the scaffold in
`foundation-log.md`; `docs touch foundation-log.md` (the expanded empty-prefix
operand).

### Phase 9 — DoR and hand-off

DoR doc authored per the template above. Mechanical gate:

```sh
docs check . --stale 14   # exit 0
```

Wizard ticks every checkbox after qualitative review, flips every front-half
doc to `Lifecycle: active`, and updates `status.md` with the
foundation-complete narrative and relative tracker link. It then touches all
changed docs in one qualified batch; for this empty-prefix example:

```sh
docs touch charter.md scope-and-constraints.md stakeholders.md \
  options-comparison.md architecture.md milestone-plan.md \
  env-and-tooling.md data-plan.md test-strategy.md \
  documentation-plan.md definition-of-ready.md status.md
docs index .
docs check . --stale 14   # exit 0
```

The tracker and status paths were touched together before index/check.
Hand-off check:

```sh
$ docs list --root . --project link-checker --lifecycle active
charter.md                    charter      active
scope-and-constraints.md      spec         active
stakeholders.md               reference    active
options-comparison.md         decision     active
architecture.md               sketch       active
decision-log.md               log          active
milestone-plan.md             plan         active
env-and-tooling.md            runbook      active
data-plan.md                  plan         active
test-strategy.md              outline      active
documentation-plan.md         plan         active
foundation-log.md             log          active
risks.md                      log          active
followup-log.md               log          active
feedback-log.md               log          active
definition-of-ready.md        reference    active
status.md                     status       active
```

17 docs, all `active`, every `Related:` link resolves,
`docs check` exit 0. Ready for the `use-cases` stage, then
`create-milestones`.
