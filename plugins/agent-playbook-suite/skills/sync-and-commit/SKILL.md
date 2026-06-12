---
name: sync-and-commit
description: Verify the work is consistent, complete, and accurate; sync the docs tree; commit and push. Use at the end of a TDD phase or step (especially when called from ship-milestone), or whenever the operator wants to wrap up in-flight work. Reads CLAUDE.md, AGENTS.md, or equivalent project context for conventions; runs `docs check` against the docs tree; never bypasses git hooks; never pushes to `main`.
---

# sync-and-commit

End-of-step wrap-up. Verify the work, sync the docs tree to
reality, and commit (and push, when on a feature branch with a
remote).

Optionally pairs with [`docs-cli`](https://github.com/ArtRichards/docs-cli)
(use the newest release) for the `Lifecycle:` / `Role:` /
`Updated:` metadata convention and atomic multi-file
`docs touch <file>...` used by `project-foundation`,
`create-milestones`, and `ship-milestone`. If the project does not
use docs-cli, the docs-side checks are skipped and only the
project's own verification commands run.

## Step 1 — Read project context

Read the project-root agent context (`CLAUDE.md`, `AGENTS.md`, or
equivalent) for:

- The docs-tree location (if any) and which artifacts live in it.
- Build/test/quality commands.
- Commit message conventions.
- Branch conventions (especially: which branches are safe to
  push to vs. which require operator review).
- Any project-specific rules.

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
- Selected tests/checks pass, or unrelated known failures are documented.

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

## Step 3 — Documentation sync

If the project uses docs-cli, sync the docs tree:

1. Update the milestone's task plan + implementation log via body
   edits (phase section, progress-table cell, checklist tick).
2. Update `status.md`'s "Current Phase" or "Current Milestone"
   section.
3. `docs touch <slug>.md <slug>-impl.md status.md` (and any other
   docs whose body content changed this step).
4. `docs index <root>` to regenerate the marker block.
5. `docs check <root> --stale 14`. Treat a docs-check failure as
   blocking. Exit 2 always means stop and fix the lifecycle drift /
   broken refs before committing. If the project treats stale docs or
   warnings as failures, fail closed on those too.

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

### Risk-aware quality gate

Lite:

- format/lint/type/build where configured;
- visible product tests for changed behavior;
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
- recorded operator approval after the Phase 4 RED baseline before
  phases 5-10 or final sync, unless an explicit operator-approved
  exception is logged;
- explicit approval for any skipped High-risk deep gate.

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

### Verification report

Before committing, prepare a verification report in the operator
response and, when the project has one, the implementation log or
quality log:

```markdown
## Verification report

- Risk level:
- Fast gate commands run:
- Deep gate commands run:
- Deferred/not configured gates:
- Visible test result:
- Hidden/generalization result:
- Coverage:
- Mutation:
- Property/stateful:
- Fuzz:
- Benchmark/security/schema/migration:
- Mock audit:
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
- selected visible product-test or explicit-check failure;
- configured build, lint, type, format, or package gate failure;
- ambiguous or potentially High risk level that cannot be inferred safely;
- missing contract/test evidence needed to judge Standard or High-risk work;
- unapproved skipped High-risk deep gates selected for the milestone;
- unauthorized visible-test or selected-check weakening, deletion, or skip;
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
