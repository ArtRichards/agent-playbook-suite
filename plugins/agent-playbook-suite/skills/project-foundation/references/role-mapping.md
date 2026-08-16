# Role and Lifecycle Mapping

Every foundation artifact maps to a controlled-vocab `Role:` and
a starting `Lifecycle:` value. Pass these to `docs new` on every
authoring call. Lifecycle transitions happen by editing the
`Lifecycle:` value in the metadata block, then touching the qualified path.

Every basename below lives under the single `<project-path>` established by
the foundation playbook: empty for a dedicated root, or a trailing-slash
root-relative prefix such as `specs/payments/` in a shared root. Qualify every
docs-cli document operand and `Related:` target with it. Keep Markdown body
links between docs in the same project directory relative. Run docs-cli from
the docs root; `--root` does not rebase relative document operands.

Assumes docs-cli 2.0 or newer — `Lifecycle:` is the controlled-vocab
field name, `docs new --body-from` is available, multi-file
`docs touch <project-path><file>.md...` is atomic, and recognized reciprocal
relationships are managed with `docs relate`.

## Foundation artifacts

| Slug (kebab-case) | Role | Starts as | Graduates to | Notes |
|---|---|---|---|---|
| `charter` | `charter` | `draft` | `active` when DoR is green | One per project. Frozen prose. |
| `scope-and-constraints` | `spec` | `draft` | `active` | The boundary contract. |
| `stakeholders` | `reference` | `draft` | `active` | Evergreen lookup. |
| `options-comparison` | `decision` | `draft` | `active` after choice; `superseded` if revisited | One per design choice. |
| `architecture` | `sketch` | `draft` | `reference` once authoritative | Early-design diagrams; promote when stable. |
| `decision-log` | `log` | `active` | stays `active` for the project's life | Ongoing append; dated H2 entries. |
| `milestone-plan` | `plan` | `draft` | `active` | Canonical `Order \| Milestone \| State \| Depends on \| Notes` tracker. Per-milestone task plans belong to `create-milestones`. |
| `env-and-tooling` | `runbook` | `draft` | `active` | Commands to run — operational. |
| `data-plan` | `plan` | `draft` | `active` | Forward-looking strategy. |
| `test-strategy` | `outline` | `draft` | `active`; → `spec` if it becomes the canonical contract | Detail fills in as implementation proceeds. |
| `documentation-plan` | `plan` | `draft` | `active` | Covers docs outside the docs tree. |
| `risks` | `log` | `active` | stays `active` | Risk Register — dated entries with mitigation/owner. |
| `followup-log` | `log` | `active` | stays `active` | Engineering follow-ups: adequacy gaps, skipped deep gates, deferred test work. Single home for open items; milestone docs reference open entries; an entry is removed and its content rehomed when a milestone incorporates it. Tight entry shape (item, source milestone, risk, owner, status) embedded in the doc body. |
| `feedback-log` | `log` | `active` | stays `active` | Operator feedback, ideas, and scope thoughts as dated intake entries (template embedded in the doc body). Same single-home rule as the follow-up log. |
| `foundation-log` | `log` | `active` | `archived` only when the project ends | Front-half decision/note audit trail. |
| `definition-of-ready` | `reference` | `draft` | `active` when green; `superseded` if rewritten | A checklist that links out. |
| `status` | `status` | `active` | stays `active` | Curated narrative and links. It does not duplicate tracker rows or independently declare schedule or next work. |

## Files outside the docs tree

A few files this skill writes live at the project repo root, not
in the docs tree. They have no `Lifecycle:` metadata, don't
appear in `INDEX.md`, and aren't validated by `docs check`:

| Path | Owner | Notes |
|---|---|---|
| `<repo-root>/CLAUDE.md` | scaffolded or extended at Phase 8 | Project context for Claude Code agents. Renders [`claude-md-template.md`](claude-md-template.md) with derived slots. If pre-existing, the skill writes `CLAUDE-additions.md` next to it for operator review rather than overwriting. |
| `<repo-root>/AGENTS.md` | scaffolded or extended at Phase 8 when Codex or a compatible host is used | Project context for Codex-style agents. Uses the same substantive sections as `CLAUDE.md`; if pre-existing, the skill writes `AGENTS-additions.md` next to it for operator review rather than overwriting. |

## Per-milestone artifacts (owned by create-milestones)

For reference — these are authored by the `create-milestones`
skill, not this one. Documented here so the foundation wizard
can hand off cleanly.

The project-unique lowercase kebab-case semantic slug in the canonical
tracker is stable identity. Reordering changes only `Order`; after the row's
first transition to `active`, the slug is frozen. Existing numeric or
hierarchical identities remain valid during migration and are not mass-renamed.

Physical files use `<artifact-stem>`: the semantic slug in a dedicated root,
or a project-qualified basename such as `<project>-<milestone-slug>` in a
shared root. This is separate from tracker identity because docs-cli flattens
archived paths into `archive/<date>/`; every new stem and its two companion
basenames must therefore be root-globally unique. A materialized activated or
historical legacy row keeps the stem already recorded by its tracker link/path.

| Slug pattern | Role | Starts as | Graduates to | Notes |
|---|---|---|---|---|
| `<project-path><artifact-stem>` | `milestone` | `draft` | `active` while in flight; `archived` on completion | One materialized task plan per linked tracker row. Its tracker link text remains the semantic slug. Archive only the exact approved artifact set. |
| `<project-path><artifact-stem>-impl` | `log` | `active` | `archived` with its milestone | Phase-by-phase TDD log. Paired via free-form `Related: pairs-with` using qualified targets. |
| `<project-path><artifact-stem>-test-matrix` | `spec` | `active` | `archived` with its milestone | Contract-to-test matrix covering risk level, visible tests, hidden/generalization categories, adequacy checks, mock audit, gate commands, and result summaries. Paired via free-form `Related: pairs-with` using qualified targets. |
| `<project-path>quality/quality-log-<artifact-stem>` | `log` | `active` | project-specific | Optional long-lived companion for generated quality reports and benchmark/mutation/security summaries. Its basename starts with `quality-log-` so the milestone archive scope `'<project-path><artifact-stem>-*'` cannot select it. Deferred deep-gate follow-ups live in `<project-path>followup-log.md`, not here. Link generated reports with qualified `Related:` targets when they live inside the docs tree. |

## Lifecycle vocabulary (built-in)

| Value | When |
|---|---|
| `draft` | Being written; not authoritative. |
| `active` | Current; in use; source of truth. |
| `blocked` | A document artifact paused on an external condition. Pair it with `blocked-by` through `docs relate add`; milestone scheduling derives blocking from that live edge and does not add a tracker `blocked` state. |
| `done` | Complete; intentionally kept in the active tree as evergreen reference. Rarely needed in foundation work (most graduate to `active`). |
| `archived` | Complete and moved under `archive/<date>/` by `docs archive`. |
| `superseded` | Replaced by another doc. Pair with `Related: superseded-by: …`. |

## Relationship verbs (the inter-doc graph)

`Related:` entries use the form
`<verb>: <project-path><target>.md`. `docs
check` validates that every target resolves to a file under the
docs root.

| Verb | Use it when |
|---|---|
| `pairs-with` | Bidirectional sibling (charter ↔ scope; status ↔ milestone-plan). |
| `child-of` / `parent-of` | Hierarchical (a milestone is `child-of` the milestone-plan). |
| `implements` | This doc realises that spec/charter (architecture `implements` charter). |
| `spec-of` | This doc specifies that thing (rare in foundation work). |
| `supersedes` / `superseded-by` | Replacement (a new DoR `supersedes` the old one). |
| `precedes` / `follows` | Derived navigation between materialized milestones in adjacent tracker `Order` cohorts. |
| `depends-on` / `required-by` | A durable prerequisite between materialized milestones, synchronized with tracker `Depends on`. |
| `blocks` / `blocked-by` | A live condition preventing work now; separate from a durable dependency. |
| `decision` | Points at the decision doc that justifies this one. |
| `references` | Weakest form — "see also." |

**Three verb pairs are reciprocal at docs 2.0** — `precedes`/`follows`,
`depends-on`/`required-by`, and `blocks`/`blocked-by`. Each half must be
declared on both endpoints; a one-sided edge is a hard `missing-inverse`
error and `docs check` exits 2. Write them with
`docs relate add <project-path><source>.md <verb> <project-path><target>.md`,
which edits both ends in one call, and remove them with the corresponding
`docs relate remove`. If either endpoint is
archived, include the required one-line `--reason`; hand-editing one side
is what produces the error. The other verbs in the table above
(`pairs-with`, `child-of`/`parent-of`, `supersedes`/`superseded-by`,
`implements`, `spec-of`, `decision`, `references`) stay free-form with no
reciprocal validation.

## Role graduation

Roles such as `sketch`, `outline`, and `implementation` describe
early-stage docs whose detail fills in over time. The wizard starts
architecture and test-strategy docs at these light roles and graduates
them once the doc becomes authoritative.

Graduation is a `Role:` field edit, then
`docs touch <project-path><file>.md`.
Same file, same slug, same history.

- **`sketch` → `reference`** when the architecture doc starts
  being cited as authoritative, not as a working sketch.
  Usually around DoR.
- **`outline` → `spec`** when the test strategy graduates from
  "what we plan to validate" to "what the product-test and
  explicit-check contract guarantees." Often during Phase 2 of
  the first milestone.
