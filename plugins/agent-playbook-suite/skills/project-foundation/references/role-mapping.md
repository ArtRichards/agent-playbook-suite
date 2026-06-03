# Role and Lifecycle Mapping

Every foundation artifact maps to a controlled-vocab `Role:` and
a starting `Lifecycle:` value. Pass these to `docs new` on every
authoring call. Lifecycle transitions happen by editing the
`Lifecycle:` value in the metadata block, then `docs touch`.

Assumes a current docs-cli release — `Lifecycle:` is the
controlled-vocab field name, `docs new --body-from` is available,
multi-file `docs touch <file>...` is atomic, and the role vocab
includes `sketch`, `outline`, `implementation`, `template`,
`example`, `memo`, `brief` (added in M7).

## Foundation artifacts

| Slug (kebab-case) | Role | Starts as | Graduates to | Notes |
|---|---|---|---|---|
| `charter` | `charter` | `draft` | `active` when DoR is green | One per project. Frozen prose. |
| `scope-and-constraints` | `spec` | `draft` | `active` | The boundary contract. |
| `stakeholders` | `reference` | `draft` | `active` | Evergreen lookup. |
| `options-comparison` | `decision` | `draft` | `active` after choice; `superseded` if revisited | One per design choice. |
| `architecture` | `sketch` | `draft` | `reference` once authoritative | Early-design diagrams; promote when stable. |
| `decision-log` | `log` | `active` | stays `active` for the project's life | Ongoing append; dated H2 entries. |
| `milestone-plan` | `plan` | `draft` | `active` | Per-milestone task plans (`m1.md` etc.) belong to `create-milestones`. |
| `env-and-tooling` | `runbook` | `draft` | `active` | Commands to run — operational. |
| `data-plan` | `plan` | `draft` | `active` | Forward-looking strategy. |
| `test-strategy` | `outline` | `draft` | `active`; → `spec` if it becomes the canonical contract | Detail fills in as implementation proceeds. |
| `documentation-plan` | `plan` | `draft` | `active` | Covers docs outside the docs tree. |
| `risks` | `log` | `active` | stays `active` | Risk Register — dated entries with mitigation/owner. |
| `foundation-log` | `log` | `active` | `archived` only when the project ends | Front-half decision/note audit trail. |
| `definition-of-ready` | `reference` | `draft` | `active` when green; `superseded` if rewritten | A checklist that links out. |
| `status` | `status` | `active` | stays `active` | Curated narrative. INDEX.md carries the machine view. |

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

| Slug pattern | Role | Starts as | Graduates to | Notes |
|---|---|---|---|---|
| `m<N>` (or `m<N>-<topic>`) | `milestone` | `draft` | `active` while in flight; `archived` via `docs archive --cascade` when complete | One per milestone in the milestone-plan. |
| `m<N>-impl` | `log` | `active` | `archived` (cascaded with its milestone) | Phase-by-phase TDD log. Paired via `Related: pairs-with`. |
| `m<N>-test-matrix` | `spec` | `active` | `archived` (cascaded with its milestone) | Contract-to-test matrix covering risk level, visible tests, hidden/generalization categories, adequacy checks, mock audit, gate commands, and result summaries. Paired via `Related: pairs-with`. |
| `quality/m<N>-quality-log` | `log` | `active` | project-specific | Optional companion for generated quality reports, benchmark/mutation/security summaries, and deferred deep-gate follow-ups. Link generated reports with `Related:` when they live inside the docs tree. |

## Lifecycle vocabulary (built-in)

| Value | When |
|---|---|
| `draft` | Being written; not authoritative. |
| `active` | Current; in use; source of truth. |
| `blocked` | Paused on an external dependency. Pair with `Related: blocked-by: …`. |
| `done` | Complete; intentionally kept in the active tree as evergreen reference. Rarely needed in foundation work (most graduate to `active`). |
| `archived` | Complete and moved under `archive/<date>/` by `docs archive`. |
| `superseded` | Replaced by another doc. Pair with `Related: superseded-by: …`. |

## Relationship verbs (the inter-doc graph)

`Related:` entries use the form `<verb>: <target.md>`. `docs
check` validates that every target resolves to a file under the
docs root.

| Verb | Use it when |
|---|---|
| `pairs-with` | Bidirectional sibling (charter ↔ scope; status ↔ milestone-plan). |
| `child-of` / `parent-of` | Hierarchical (a milestone is `child-of` the milestone-plan). |
| `implements` | This doc realises that spec/charter (architecture `implements` charter). |
| `spec-of` | This doc specifies that thing (rare in foundation work). |
| `supersedes` / `superseded-by` | Replacement (a new DoR `supersedes` the old one). |
| `blocked-by` | Pause causation (a doc `blocked-by` an open question). |
| `decision` | Points at the decision doc that justifies this one. |
| `references` | Weakest form — "see also." |

## Role graduation

M7 introduces `sketch`, `outline`, `implementation`, and other
roles for early-stage docs whose detail fills in over time. The
wizard starts most architecture and test-strategy docs at these
light roles and graduates them once the doc becomes
authoritative.

Graduation is a `Role:` field edit, then `docs touch <file>`.
Same file, same slug, same history.

- **`sketch` → `reference`** when the architecture doc starts
  being cited as authoritative, not as a working sketch.
  Usually around DoR.
- **`outline` → `spec`** when the test strategy graduates from
  "what we plan to test" to "what the test contract
  guarantees." Often during Phase 2 of the first milestone.
