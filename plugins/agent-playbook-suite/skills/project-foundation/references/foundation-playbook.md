# Foundation Playbook

The step-by-step procedure the `project-foundation` skill drives.
Short trigger lives in [`../SKILL.md`](../SKILL.md); substance
lives here so SKILL.md can stay small.

Pair this with [`role-mapping.md`](role-mapping.md) for the
artifact-to-role-and-lifecycle table — every `docs new` call
below uses the role and starting `Lifecycle:` from that table.

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
     project? When it is appropriate, default to placing the
     foundation under `<found-root>/specs/<project-slug>/`;
     otherwise ask whether to bootstrap a new dedicated root.
   - **No root found** → inspect the existing directories before
     suggesting a location. Classify any candidate documentation
     directory as internal vs public/customer-facing before
     reusing it: public signals include a documentation builder,
     a tool that builds docs for customers, or generated customer
     docs. Never place internal specs inside a public
     documentation surface, or in site/publishing directories
     such as `site/`, `website/`, `pages/`, or `public/`, even
     when they contain Markdown.
     - `docs/` exists and is internal → prefer
       `<repo-root>/docs/specs/<project-slug>/` when that fits
       the layout. Single-project repos may use the flat
       `docs/specs/` layout (no subdirectory).
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

2. **Bootstrap the new root** by:
   - Creating the directory.
   - Copying [`docs-toml-template.toml`](docs-toml-template.toml)
     to `<root>/.docs.toml` and substituting the project slug
     into `[project] name`.
   - Running `docs check <root>` once (empty tree always passes).

3. **Resolve the project slug** — kebab-case, lowercase. Used by
   `--project` on every `docs new` call. Record the chosen
   working directory; every subsequent `docs new` runs from
   there.

4. **Create the five living docs** before any phase question:
   `status.md` (`Role: status`, `Lifecycle: active`),
   `foundation-log.md` (`Role: log`, `Lifecycle: active`),
   `risks.md` (`Role: log`, `Lifecycle: active`),
   `followup-log.md` (`Role: log`, `Lifecycle: active`), and
   `feedback-log.md` (`Role: log`, `Lifecycle: active`). These
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

If the operator asks for it, launch a thorough investigation of
the entire product's foundation — architecture, module
boundaries, dependencies, data flows, test coverage, operational
surfaces — before or during foundation work. Record the findings
in `foundation-log.md` (and `architecture.md` once it exists) so
the investigation outlives the session.

## Authoring docs without harness friction

Every `docs new` call in this playbook uses the M8 `--body-from`
flag so metadata block and body land in a single Bash call, with
no Read-before-Write round trip:

```sh
docs new <role> <slug> --project <p> --title "<H1>" --body-from - <<'EOF'
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

`Related:` is the one metadata field this skill extends after
`docs new`. Use a careful Edit on the metadata block to add
`- <verb>: <target.md>` lines under the existing `Related:`
list (or to insert a `Related:` block if one wasn't scaffolded).
Then `docs touch <file>` to bump `Updated:`.

## Phase 0 — Intake & Alignment

Ask:

- What problem are you trying to solve?
- Who is the target user?
- What is the measurable success metric?
- What are the non-goals?

Author:

```sh
docs new charter charter --project <p> --title "<Project>: Charter" --body-from - <<'EOF'
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

Author `docs new spec scope-and-constraints` with body sections:
**In scope**, **Out of scope**, **Assumptions**, **Constraints**,
**Open questions**. Add `Related: pairs-with: charter.md`.

## Phase 2 — Stakeholders & Interfaces

Ask:

- Who are the stakeholders?
- What external systems/APIs will this integrate with?
- What data will be consumed or produced?
- Who owns each interface?

Author `docs new reference stakeholders` with body sections:
**Stakeholders**, **Interfaces (consumed)**, **Interfaces
(produced)**, **Data flows**, **Ownership**. Add
`Related: pairs-with: charter.md`.

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

Author three docs:

1. `docs new decision options-comparison` —
   `Lifecycle: draft` while the choice is open. Body:
   **Context**, **Options considered** (one H3 per option, with
   pros/cons), **Selected approach**, **Rationale**. Flips to
   `active` once decided.
2. `docs new sketch architecture` — `Lifecycle: draft`. Body:
   **Shape**, **Data flow**, **Integration points**. Add
   `Related: implements: charter.md`, `Related: pairs-with:
   options-comparison.md`. Graduates to `Role: reference` when
   authoritative.
3. `docs new log decision-log` — `Lifecycle: active`. Ongoing
   log of choices made during the project; first entry
   summarises the Phase 3 architecture choice. Append dated
   `## YYYY-MM-DD — <one-line>` entries as decisions accumulate.

## Phase 4 — Delivery Strategy & Milestones

Ask:

- What are the major milestones (3-5 checkpoints)?
- What does each deliver?
- What are the dependencies between them?
- Where are the demo/review checkpoints?

Author `docs new plan milestone-plan` with body sections:
**Sequencing** (ASCII flow diagram if useful), **Milestones**
(one H3 per milestone — `### M1 — <title>`, with goal +
dependencies + demo checkpoint), **Buffer notes**.

Add `Related: implements: charter.md`,
`Related: pairs-with: architecture.md`. Per-milestone task
plans (`m1.md`, etc.) belong to `create-milestones`, not here.

## Phase 5 — Environment & Tooling

Ask:

- What runtime/language/framework will be used?
- What are the build commands?
- What are the test commands?
- What access/credentials are needed?
- Any blockers to getting started?

Author `docs new runbook env-and-tooling` with body sections:
**Runtimes**, **Build commands**, **Test commands**,
**Access/credentials**, **Blockers**. `runbook` because the
content describes commands to run, not concepts to think about.

## Phase 6 — Data & Compliance

Ask:

- What data sources will be used?
- Synthetic or real data for development/testing?
- Data quality expectations?
- Privacy/security/compliance constraints?

Author `docs new plan data-plan` with body sections:
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
or product first, then propose a level per area with one-line
reasoning and confirm with the operator. Default to Standard for
ordinary product or workflow changes; reserve Lite for docs,
internal/admin work, and low-blast-radius changes. Propose High
only with an explicit reason from the shared quality model's High
triggers (auth, billing, security, privacy, data integrity,
migrations, concurrency, incident response, public APIs,
performance-sensitive core paths) — and get the operator's
explicit approval before recording High. Record the agreed level
and its reasoning in the risk table.

Author `docs new outline test-strategy` with body sections:
**Validation taxonomy**, **Risk levels**, **Fast PR gate**,
**Deep/nightly/release gate**, **Hidden-test policy**,
**Mock policy**, **Human approval triggers**, and **Coverage /
adequacy metrics**. `outline` (not `spec`) because the detail fills
in as implementation proceeds; graduates to `spec` later if it
becomes the canonical test contract.

Use this starter shape:

```markdown
## Validation taxonomy

- Product tests:
- Non-product checks:
- Functional tests:
- Technical tests:
- Cross-cutting adequacy tests:

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

Author `docs new plan documentation-plan` with body sections:
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

Run this **after** authoring `documentation-plan.md`:

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
   conventions.
   - **All fixed sections present** -> the file is good as-is. Note
     that in `foundation-log.md`; do not write an additions file.
   - **One or more missing** -> write `CLAUDE-additions.md` and/or
     `AGENTS-additions.md` next to the existing file with the proposed
     additions (rendered, with derived slots filled). Show the user the
     path and a short summary of what's proposed; let them review and
     merge by hand.

4. **Add context files to `documentation-plan.md`'s Required docs
   table** — name `CLAUDE.md` and/or `AGENTS.md`, role "project
   context for coding agents", owner the project owner, cadence
   "updated when skill ecosystem, quality gates, or tooling changes."

5. **Note the action in `foundation-log.md`** — one of:
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
docs new reference definition-of-ready --project <p> --title "<Project>: Definition of Ready" --body-from - <<'EOF'
Gate-check before implementation begins. Implementation does not start until every item is green.

## Foundation checklist

- [ ] **Charter** states problem, user, success metric, non-goals. → [charter.md](charter.md)
- [ ] **Scope, assumptions, constraints captured.** → [scope-and-constraints.md](scope-and-constraints.md)
- [ ] **Stakeholders/interfaces noted with owners.** → [stakeholders.md](stakeholders.md)
- [ ] **Architecture option chosen; decision recorded.** → [options-comparison.md](options-comparison.md), [architecture.md](architecture.md)
- [ ] **Milestones and dependencies mapped; timeline feasible.** → [milestone-plan.md](milestone-plan.md)
- [ ] **Environment/tooling validated; access unblocked.** → [env-and-tooling.md](env-and-tooling.md)
- [ ] **Data plan set; compliance/privacy constraints clear.** → [data-plan.md](data-plan.md)
- [ ] **Test strategy outline covers critical paths and fixtures.** → [test-strategy.md](test-strategy.md)
- [ ] **Risk level is assigned for each milestone or project area.** → [test-strategy.md](test-strategy.md)
- [ ] **Fast PR gate commands are documented.** → [test-strategy.md](test-strategy.md)
- [ ] **Deep/nightly/release gate commands are documented or explicitly marked not applicable.** → [test-strategy.md](test-strategy.md)
- [ ] **Hidden/generalization strategy is documented without exposing private cases.** → [test-strategy.md](test-strategy.md)
- [ ] **Contract-to-test matrix template exists.** → [test-strategy.md](test-strategy.md)
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

Add `Related: pairs-with: charter.md`,
`Related: pairs-with: milestone-plan.md`,
`Related: pairs-with: status.md`.

**Mechanical gate.** Before declaring green:

```sh
docs check <root> --stale 14
```

Exit-code verdict:

- `0` → mechanically clean. Apply qualitative review (every
  checkbox justified, every link resolves, success metric
  measurable, residual risks logged). Flip the DoR doc and
  every front-half doc from `Lifecycle: draft` to
  `Lifecycle: active`, then `docs touch` each.
- `1` → warnings (medium-confidence inferences or stale docs).
  Review; `docs touch` if still correct, or update content.
  Re-run.
- `2` → errors (missing fields, broken refs, lifecycle/location
  drift). Fix before proceeding. DoR cannot pass.

## Completion and hand-off

When DoR flips to `active`:

1. `docs check <root> --stale 14` exit 0.
2. `docs list --lifecycle active --project <p>` shows every
   artifact in the expected state.
3. Update `status.md`'s "Current milestone" section:
   _Pending → M1 setup_.
4. `docs touch status.md`.
5. Hand off to the `use-cases` skill — it runs automatically
   after foundation completes (optional, but strongly preferred)
   to explore the primary use cases that will guide testing.
6. Then hand off to `create-milestones`.

## Worked example: link-checker

A small fictional project — a CLI that crawls a website and
reports broken links. Project slug: `link-checker`. User
invoked the skill from inside `~/code/link-checker/` (a repo
with a `docs/` directory).

### Step 0 — Bootstrap

Wizard walks up three levels, finds no `.docs.toml`. Default
suggestion (`docs/` exists → `docs/specs/`) confirmed. Then:

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
**Single source of truth for project progress. Update only this file when milestones complete or phases advance.**

## Current milestone

_Pending — foundation work in progress._

## Quick links

- [Charter](charter.md)
- [Milestone Plan](milestone-plan.md)
- [Definition of Ready](definition-of-ready.md)
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
`foundation-log.md` after each. After all eight phases, the
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

`milestone-plan.md`'s body lists three milestones:

- M1 — Fetch and parse (HTTP client, link extraction, in-memory crawl)
- M2 — Persistence (SQLite cache, resume from prior crawl)
- M3 — Reporting (HTML/JSON output, summary stats)

### Phase 8 addendum — CLAUDE.md scaffold

`~/code/link-checker/CLAUDE.md` doesn't exist (greenfield repo).
Wizard renders `claude-md-template.md` with the derived slots:

- `<project summary>`: first paragraph of `charter.md`.
- `<docs-root-path>`: `docs/specs`.
- `<project-slug>`: `link-checker`.
- Build/test commands: from `env-and-tooling.md` (e.g. `pytest`,
  `ruff check .`, `mypy`).
- Commit conventions: defaulted (no prior git history yet).

Writes to `~/code/link-checker/CLAUDE.md`; notes the scaffold in
`foundation-log.md`; `docs touch foundation-log.md`.

### Phase 9 — DoR and hand-off

DoR doc authored per the template above. Mechanical gate:

```sh
docs check . --stale 14   # exit 0
```

Wizard ticks every checkbox after qualitative review, flips
every front-half doc to `Lifecycle: active`, then:

```sh
docs touch charter.md scope-and-constraints.md stakeholders.md \
  options-comparison.md architecture.md milestone-plan.md \
  env-and-tooling.md data-plan.md test-strategy.md \
  documentation-plan.md definition-of-ready.md
docs index .
docs check . --stale 14   # exit 0
```

Updates `status.md`'s "Current milestone" to
`Pending → M1 setup`, `docs touch status.md`. Hand-off check:

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
