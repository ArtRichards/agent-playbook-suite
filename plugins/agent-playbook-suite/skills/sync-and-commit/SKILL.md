---
name: sync-and-commit
description: Verify work, semantic milestone-tracker state, and any already-applied ship-milestone archive closeout; sync active docs; commit and push. Use at a TDD phase or step boundary, especially when called from ship-milestone, or to wrap up in-flight work. Never performs milestone archival, edits or touches archived milestone docs, bypasses hooks, or pushes to main.
---

# sync-and-commit

End-of-step wrap-up. Verify the work, sync the docs tree to reality, and commit
(and push, when on a feature branch with a remote).

Optionally pairs with [`docs-cli`](https://github.com/ArtRichards/docs-cli) (use
the newest release) for managed metadata and atomic multi-file `docs touch`
used by `project-foundation`, `create-milestones`, and `ship-milestone`. Without
docs-cli, skip docs-side checks and run only project verification.

For a suite milestone tracker, read and enforce the shared contract at
[`../_shared/references/milestone-tracker.md`](../_shared/references/milestone-tracker.md).
Verify the row and handoff; never select/claim/complete work or archive artifacts.

For suite-managed work, `<project-path>` is a concrete docs-root-relative prefix:
empty in a dedicated root, otherwise ending in `/`. Branch roots use
`<project-branch-prefix><slug>/...`: prefix `<project>/` in repositories with
multiple suite projects and leave it empty only for one project-wide namespace;
Project slugs are repository-globally unique.
Path-free `<artifact-stem>` identifies physical files; semantic `<slug>` is
tracker/branch identity. Tracker/status use `<project-path>milestone-plan.md`
and `status.md`; artifacts add `.md`, `-impl.md`, and `-test-matrix.md` to the stem.

With docs-cli, resolve and change to the docs root before every operation. `--root`
does not rebase relative `FILE`, `SOURCE`, or `TARGET` operands or replace that
`cd`; keep operands root-relative and use `docs index .` / `docs check .` there.

Before verification, read the shared tracker reference in full. If required but
absent, stop before Step 1, explain the suite installation is incomplete, and
recommend reinstall/update. Never reconstruct the contract from project/caller evidence.

Read the shared operator-interaction rules at
[`../_shared/references/operator-interaction.md`](../_shared/references/operator-interaction.md)
before asking anything. If absent, still explain the question's origin, why now,
practical effect, and a grounded or labeled-hypothetical example; recommend when
evidence supports it.

For explicit `ship-milestone` Step 0, 1, or 2, also read and enforce the
[`ship-milestone` step review protocol](../ship-milestone/references/review-protocol.md).
Do not infer it for direct use or Step 3; Step 3 must provide the closeout packet
below. Fail closed rather than repair/repeat its destructive operation.

## Step 1 — Read project context

Read the project-root agent context (`CLAUDE.md`, `AGENTS.md`, or
equivalent) for:

- The docs-tree location (if any) and which artifacts live in it.
- Build/test/quality commands.
- Commit message conventions.
- Branch conventions (especially: which branches are safe to
  push to vs. which require operator review).
- Any project-specific rules.
- The canonical `<project-path>milestone-plan.md` and
  `<project-path>status.md` roles when this is a
  suite-managed milestone project.

If no project context file exists, fall back to inspecting recent
`git log --oneline` for commit style and `package.json` /
`pyproject.toml` / `Makefile` for commands.

## Step 2 — Technical verification

Quick sanity pass: read through the diff. Anything obviously
wrong or incomplete? Then walk the checklist.

### 2a. Code completeness

For the phase/step just completed, verify:

- All required functionality is implemented (check against the
  milestone spec).
- No placeholder code, `TODO`/`FIXME` comments, or
  "implement later" stubs left behind.
- No commented-out code that should be removed.
- All edge cases from the spec are handled.
- Error handling is in place where needed.
- No dead code or unused imports.

### 2b. Test completeness

- Validation is classified as product tests or explicit non-product
  checks. Planning, documentation, handoff, and workflow checks are
  not in default product-test discovery unless they define shipped
  behavior.
- Relevant unit tests for new behavior where unit boundaries are useful.
- Integration tests for meaningful cross-component interactions.
- E2E tests for user-facing flows when the milestone owns the flow.
- Important edge cases and error paths covered.
- Selected tests/checks are in the expected phase state. Ordinarily they pass.
  With explicit `ship-milestone` Step 1 context, product tests instead remain
  RED for the intended contract reason while positive verification commands
  and all unrelated selected checks pass.

### 2c. Type safety and linting

Run the project's configured quality gate. If project context is absent, infer
the smallest useful equivalent from files such as `Makefile`, `package.json`, or
`pyproject.toml` (common shape: `make format && make lint && make typecheck &&
make test`):

- Typecheck clean.
- Lint clean.
- No `any`/`unknown` types introduced without justification.

### 2d. Code consistency

- Naming, file organization, error handling, logging, and
  output formats match existing patterns in the codebase.
- The milestone's taste anchors (when recorded) are honored:
  reference modules matched, in-project libraries reused, no new
  dependency or hand-rolled equivalent without a logged decision.
- Abstraction level and comment/docstring density match the
  surrounding code.

### 2e. Accuracy — code vs. spec

- Implementation matches the milestone spec: signatures, schemas,
  exit codes, output formats, field names, constants, thresholds.
- Where code and spec disagree, decide which is wrong. Stale spec
  → fix the spec. Code bug → fix the code. Intentional divergence
  that changes intent → surface to the operator rather than
  silently accepting.

### 2f. Integration points

- Interfaces between components are correct.
- Shared types/contracts updated consistently.
- No accidental breaking changes to existing APIs (or
  intentionally-breaking changes are documented).

### 2g. Semantic milestone tracker

When the change belongs to a tracked milestone, identify exactly one row from
the explicit caller context and verify it against the shared tracker contract:

- project, concrete project path, artifact stem, semantic slug, tracker link,
  project-qualified primary/companions using that path and stem, and
  `<project-branch-prefix><slug>/...` branch root agree;
- Steps 0-2 use the same `active` row and never rename it, derive order from the
  slug, or mark it complete;
- explicit Step 3 closeout uses that same row in `complete` state after archive;
- Order, dependencies, Notes, and recognized relationship pairs are coherent;
  and
- `<project-path>status.md` links to the tracker as narrative context without
  duplicating rows or independently selecting next work.

Do not silently fix an identity, order, state, or dependency mismatch during
sync. Stop with the exact row and conflicting evidence.

## Step 3 — Documentation sync

If the project uses docs-cli and this is not explicit ship Step 3 closeout,
sync the active docs tree:

1. Resolve every target's lifecycle first. Never edit or `docs touch` an
   archived milestone document; stop if ordinary sync would require that.
2. Update the milestone's task plan + implementation log via body
   edits (phase section, progress-table cell, checklist tick).
3. Update the resolved `<project-path>status.md` narrative only when reality
   changed; never copy tracker rows or name next work independently.
4. From the resolved docs root, run
   `docs touch <project-path><artifact-stem>.md
   <project-path><artifact-stem>-impl.md <project-path>status.md` using the
   resolved concrete paths (plus any other active docs whose body changed).
5. `docs index .` to regenerate the marker block.
6. `docs check . --stale 14`. Treat a docs-check failure as
   blocking. Exit 2 always means stop and fix the lifecycle drift /
   broken refs before committing. If the project treats stale docs or
   warnings as failures, fail closed on those too.

For explicit ship Step 3 closeout, do not edit or touch the now-archived task
plan, implementation log, test matrix, or any other archived milestone
companion. The retained closeout writer has already updated the tracker and
status, regenerated the index, and run the check. Verify that handoff and rerun
`docs check . --stale 14` from the resolved docs root; if it exposes drift,
stop instead of repairing
the archive. Never run `docs archive` from this skill.

If the project does NOT use docs-cli, update whatever status /
implementation log / changelog the project's conventions require
(per project context).

Also update — if applicable — the README, architecture doc, API
reference, and operator runbook.

## Step 4 — Risk-aware quality gate

After ordinary verification and docs sync, but before commit, run the
risk-aware gate for the current milestone or change slice. Use the
project's shared quality model or test matrix when present.

Read the risk level from the milestone doc, contract/test matrix,
quality log, or project policy:

For explicit Step 3, these milestone artifacts are already archived. Read them
and the closeout packet without editing them; consume the writer's frozen gate
results rather than adding a post-archive record.

- If the risk level is missing, infer a conservative level from the change type
  and log the assumption; pause only when the risk is ambiguous or potentially
  High.
- For Standard or High risk, expect a contract/test matrix or equivalent
  contract-to-check notes. Stop only when the missing evidence makes the gate
  impossible to judge.
- For High-risk work in or after Phase 5, look for recorded operator approval
  from the Phase 4 RED-baseline checkpoint before committing.
  Acceptable evidence is a milestone-doc, test-matrix,
  implementation-log, or quality-log entry approving the contract,
  visible product tests or explicit non-product checks,
  hidden/generalization plan, mock policy, and selected gates. If project
  policy allows automatic continuation, log that policy instead of requiring a
  separate exception.
- Record skipped deep gates as `not configured`, `deferred with reason`,
  or `operator-approved skip`; never silently mark them green.

With explicit `ship-milestone` Step 1 context, the expected product-test state
is the reviewed Phase 4 RED baseline. Require captured output proving the tests
fail for the intended contract reason; an unexpected pass or an unrelated
failure blocks. Positive RED-verifier commands, docs checks, lint, type, build,
and other selected checks must still pass.

### Risk-aware quality gate

Lite:

- format/lint/type/build where configured;
- visible product tests in the phase's expected state;
- explicit non-product checks when selected;
- docs check if the project uses docs-cli.

Standard:

- Lite plus coverage report if already configured;
- property/stateful smoke where configured;
- hidden/generalization smoke where configured;
- security/schema smoke where configured;
- mock audit when mocks are added or expanded.

High:

- Standard plus benchmark/security/migration/rollback checks where they match
  the risk;
- mutation smoke or recorded mutation baseline where configured;
- recorded operator approval after the Phase 4 RED baseline before phases 5-10
  or any sync that contains Phase 5-10 work, unless an explicit
  operator-approved exception is logged. The Step 1 review-closure sync may
  record the reviewed RED evidence before approval; it does not authorize Step
  2;
- explicit approval for any skipped High-risk deep gate.
- explicit Step 3 code-changing simplification has one fresh-reviewer or operator
  approval bound to its frozen tree before checkpoint; `not required` needs no code change.

### Test adequacy and mock audit

Check:

- Contract file or contract section exists.
- Product tests or explicit non-product checks trace to contract
  clauses where practical.
- No visible test or selected explicit check was weakened, skipped,
  or deleted without a logged contract change.
- No code appears to branch on test literals, fixture names, or visible
  examples.
- New mocks are listed and justified.
- At least one real-path test exists for mocked behavior where
  relevant.
- Hidden/generalization categories were updated.
- `hidden_generalization_gap` is recorded if hidden-pass data is
  available.

### Taste triage

Per the shared quality model's Taste model:

- If a review returned taste findings for this step, every one is recorded in
  the implementation log's taste-triage table as **fixed** or **waived with a
  reason**. An untriaged taste finding is a fail-closed condition — do not
  commit past it.
- High-risk waivers show explicit operator approval.
- Recurring findings or repeated waivers are noted for promotion into the
  project's agent context (CLAUDE.md / AGENTS.md) or future milestones'
  taste anchors.

### Ship-milestone Step 0-2 review ledger

Apply this gate only when the caller explicitly passes all three of:

- workflow: `ship-milestone`;
- step: `0`, `1`, or `2`; and
- the exact current review-pass location as a qualified impl-log path plus unique anchor.

Before committing, verify that the ledger follows the linked review protocol:

- the named current pass and append-only predecessor chain are intact; repeated
  Step 1 work alternates `review-pass-N -> contract-rework-N -> review-pass-(N+1)`, while targeted re-review stays nested;
- one frozen base/`HEAD` packet is named for the original review pass;
- the Claude and GPT slots are both terminal (`returned` or `unavailable`),
  with host-reported effective provider/model for a returned review (or an
  explicit note that the host did not expose the exact model) or a concrete
  unavailable reason;
- at least one provider-family review returned, and an unavailable family was
  not replaced by a second reviewer from the available family;
- every reviewer-labeled blocker is `fixed`, `disproven` with contract/code/test
  evidence, or `operator decision` with the answer and applied evidence;
- every taste finding is fixed or waived with a reason;
- re-review is either complete with its packet and result or recorded `not
  required` with the evidence-based reason; and
- `final-sync` is `ready`, set only after every review-closure condition above
  is complete.

Fail closed if this explicit ship-step context is present and the ledger is
missing or incomplete. Outside Steps 0-2, do not require a two-provider review
ledger; run the ordinary gates in this skill.

### Ship-milestone Step 3 archive closeout

Apply this gate only when the caller explicitly passes `workflow:
ship-milestone`, `step: 3`, and all of:

- evidence mode: `normal` or `verified interruption recovery`;
- owning project, concrete project path, artifact stem, semantic slug, semantic
  branch root, and exact `<project-path>`-qualified tracker and status paths;
- canonical root-relative expected artifact set and final archive destinations;
- immutable Step 2 base SHA; clean pre-archive checkpoint SHA/tree; code-change
  classification; and tree-bound review, operator approval, or justified `not required`;
- caller-frozen archive scope, date, and reason; and
- final tracker row and docs-check result.

Evidence is mode-specific. `normal` requires the exact preview/apply commands
and docs-cli JSON. `verified interruption recovery` is allowed only after the
exact three-file move, when rerunning preview/archive is unsafe. It requires
clearly labeled reconstructed selection/apply evidence from the established
simplify-branch diff, frozen identity/scope/date/reason, all three archive paths
and lifecycle/date witnesses, rebased links, and clean `docs check .`. Never
invent missing commands/JSON or present reconstruction as captured output.
Both modes require durable checkpoint/gate evidence; archive JSON cannot replace it.

Verify without mutating archive state:

- current branch is the declared simplify branch and pre-sync `HEAD` equals the
  supplied checkpoint/tree; checkpoint equals base only with no staged change,
  otherwise its parent equals base and its commit trailers bind tree and gate;
- its tree has the three active sources and no declared archive destination;
  current diff/status has no code/unrelated-doc change, only exact archive/link
  rewrites plus tracker/status/INDEX closeout, and no unexplained untracked path;
- recompute base-to-checkpoint path classification (uncertain means code); High-risk code changes need exact-tree approval, other cases justify `not required`;
- the supplied project path is either empty or a non-absolute,
  parent-traversal-free docs-root-relative prefix ending in `/`, and the
  supplied path-free, glob-free artifact stem maps the declared expected paths
  to
  `<project-path><artifact-stem>.md`,
  `<project-path><artifact-stem>-impl.md`, and
  `<project-path><artifact-stem>-test-matrix.md` exactly;
- the caller-supplied scope is literally
  `<project-path><artifact-stem>-*`; reject a missing, unresolved, or
  unqualified value. Verify supplied values as immutable evidence—never
  synthesize, normalize, or repair the scope; semantic slug alone is never an
  archive operand;
- in `normal` mode, preview used `--cascade-dry-run`, apply did not, and both
  used the same primary, date, reason, and literal
  `--cascade-only '<archive-scope>'` value supplied in the packet; neither used
  bare/retired `--cascade`; preview selected exactly the declared three-file
  set with no tracker, status, future/paused, or cross-milestone document, and
  apply moved exactly that previewed set;
- in `verified interruption recovery` mode, the established simplify-branch
  diff reconstructs selection and movement as exactly the declared three-file
  set, excluding those same unrelated documents; its exact archive paths,
  metadata witnesses, frozen identity/scope/date/reason, rebased links, and
  clean docs check prove the move with no unexplained state or rerun;
- because docs-cli flattens archive destinations, every declared destination
  is unique across the whole docs root's archive namespace and
  does not collide with an unrelated active or archived document;
- every declared destination exists with `Lifecycle: archived` and the same
  `Archived:` date;
- no archived milestone path was edited or touched after the apply; only the
  archive operation's planned metadata/body-link rewrites are present there;
- the same tracker row changed from `active` to `complete` without changing
  Order, frozen slug, dependencies, or Notes, and its CLI-rebased archive link
  resolves; and
- `<project-path>status.md` remains a tracker-linked narrative, INDEX is
  current, `docs check .` from the resolved docs root exits 0, and the selected
  product/quality gates are GREEN.

In `normal` mode report both commands, preview/apply sets, destinations, tracker
row, and result. In recovery mode report the reconstruction and every witness,
noting that missing command/JSON evidence was not fabricated. Stop on absent,
different, or unexplained state; never re-preview, rerun archive, or edit/touch
archived docs.

### Verification report

Before committing, prepare a verification report in the operator response and,
when the project has one, the active implementation log or quality log. For
Step 3 closeout, keep the report in the operator response because those
milestone artifacts are archived and immutable to this skill:

```markdown
## Verification report

- Risk level:
- Fast gate commands run:
- Deep gate commands run:
- Deferred/not configured gates:
- Visible test result:
- Expected visible-test state (GREEN or intentional Step 1 RED):
- Hidden/generalization result:
- Coverage:
- Mutation:
- Property/stateful:
- Fuzz:
- Benchmark/security/schema/migration:
- Mock audit:
- Taste triage (findings fixed/waived; untriaged = fail):
- Ship-milestone Step 0-2 review ledger (when explicitly in scope):
- Semantic tracker row / branch identity (when in scope):
- Ship-milestone Step 3 evidence mode and preview/apply/reconstruction evidence:
- Step 3 simplify checkpoint SHA/tree and High-risk gate evidence:
- Contract/test matrix status:
- Docs check:
- Diff scope:
- Commit:
- Push:
- Operator approvals:
- High-risk RED-baseline approval:
- Remaining risks:
```

### Fail-closed behavior

Stop before commit and fix or escalate on:

- docs-check failure;
- selected visible product-test or explicit-check failure, except the verified
  intentional RED state for an explicit `ship-milestone` Step 1 call;
- an explicit Step 1 RED baseline that passes unexpectedly or fails for a
  reason other than the intended contract gap;
- configured build, lint, type, format, or package gate failure;
- ambiguous or potentially High risk level that cannot be inferred safely;
- missing contract/test evidence needed to judge Standard or High-risk work;
- unapproved skipped High-risk deep gates selected for the milestone;
- unauthorized visible-test or selected-check weakening, deletion, or skip;
- an untriaged taste finding from this step's review (no fixed or
  waived-with-reason record in the taste-triage table), or a High-risk
  taste waiver without operator approval;
- explicit `ship-milestone` Step 0-2 caller context with a missing or incomplete
  current anchored review pass or predecessor chain, no returned provider-family
  review, or an unresolved reviewer blocker;
- tracked-milestone context whose project, semantic identity, row state,
  project path, artifact stem, companions, dependencies, or branch root
  disagree;
- explicit `ship-milestone` Step 3 context with missing/mismatched
  checkpoint/base/tree or applicable High-risk gate, post-checkpoint code or
  unexplained paths, evidence mode, project path, artifact stem, frozen scope,
  expected-path mapping, root-global uniqueness proof, or mode-appropriate
  preview/apply/reconstruction evidence; a
  non-exact artifact set; an unsafe tracker completion; a broken archive link;
  or any required post-archive edit/touch;
- unexplained new or expanded mocks in Standard or High-risk work.

Do not proceed by weakening tests, removing hooks, or relabeling a
selected gate as optional. If a configured gate cannot run, record
whether it is `not configured`, `deferred with reason`, or
`operator-approved skip`.

## Step 5 — Review changes

```sh
git status
git diff
```

Review every file that will be committed:

- No sensitive files staged (secrets, credentials, `.env`).
- Changes look correct and complete.
- Diff scope matches this phase/step — no unrelated changes.

## Step 6 — Commit and push

1. `cd` to the right repository (multi-repo projects commit to
   each separately).
2. `git add` the relevant files explicitly — never blanket-add
   with `git add -A` unless project context says otherwise.
3. Write a commit message following the project's convention
   (concise, imperative, scoped per phase when working under
   `ship-milestone`).
4. Push to remote **only** if:
   - The current branch is a feature/milestone branch (never
     `main`, never a shared branch).
   - A remote exists.
   - The project context's branch conventions allow it.

Never use `--no-verify` or otherwise bypass git hooks unless
the operator explicitly asks.
