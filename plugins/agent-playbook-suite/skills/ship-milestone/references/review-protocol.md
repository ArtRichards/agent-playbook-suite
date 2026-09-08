# Step review protocol

Use this protocol at the end of `ship-milestone` Steps 0, 1, and 2. Step 3
uses the separate simplify-and-close archive verification and does not use this
provider-review protocol.

Use `<project-path>` for the concrete docs-root-relative project prefix: empty
for a dedicated docs root, otherwise a relative path ending in `/`. Branch
roots use `<project-branch-prefix><slug>/...`, where the prefix is the
repository-globally unique `<project>/` in a multi-project repository and is
empty in a dedicated single-project repository. Keep the semantic slug as
tracker/branch identity; record the separate path-free artifact stem used by
project-qualified milestone filenames.

## Contents

- [Freeze one review packet](#freeze-one-review-packet)
- [Attempt the two provider families](#attempt-the-two-provider-families)
- [Reconcile findings after isolation](#reconcile-findings-after-isolation)
- [Re-review only when evidence requires it](#re-review-only-when-evidence-requires-it)
- [Keep a durable append-only resume ledger](#keep-a-durable-append-only-resume-ledger)

## Freeze one review packet

Start only after the retained milestone-creation or implementation agent has
returned a clean working tree and a committed review-ready `HEAD`. Freeze one
decision-relevant packet containing:

- semantic milestone slug, step, phase range, branch, exact base commit SHA, and exact
  review `HEAD` SHA;
- current `review-pass` id, its exact qualified path-plus-anchor ledger
  location, and predecessor sequence entry;
- owning project, concrete project path, artifact stem, canonical tracker path,
  the selected semantic row (Order, slug, State, Depends on), claim-checkpoint
  commit, project-qualified primary/companion paths, and semantic branch root;
- project-context, milestone, implementation-log, test-matrix, and pinned-spec
  paths;
- resolved operator decisions, risk level, QUALITY PLAN, and taste anchors;
- the selected test and quality-gate commands and their latest results; and
- the step-specific test-quality clause for the fresh-eyes prompt.

Give the packet and the same provider-neutral review prompt to every reviewer.
Invocation metadata may differ, but decision-relevant content must not. Keep
the branch unchanged until both attempts have returned or been recorded
unavailable. Reviewers are isolated: neither receives the other review's
findings.

The packet is invalid if project path is not a concrete docs-root-relative value
(empty, or non-absolute and traversal-free ending in `/`), artifact stem is
absent or contains a path separator or glob syntax,
the qualified primary/companion paths do not use that path and stem, the tracker
row is not `active`, the activated slug does not match the semantic branch root,
or `<project-path>status.md` is a second scheduler. Fix that coordination defect
before a review; do not ask reviewers to bless a divergent identity.

## Attempt the two provider families

Attempt exactly one fresh Claude-family reviewer and one fresh GPT-family
reviewer. Use only a host-provided route or an already-installed,
already-authorized local mechanism that is permitted to receive this project's
review packet. Do not install an adapter, request new credentials, or build
provider-selection machinery during a milestone run.

Count the effective provider family, not the requested label. When a route may
substitute providers and cannot confirm the effective family before launch, run
the attempts sequentially. A returned report fills its effective family slot.
For example, if a requested Claude attempt reports GPT, mark Claude
`unavailable`, use that report as the one GPT review, and do not launch another
GPT reviewer. Never count two reports from the same family.

Record the host-reported effective provider family and model for every
successful review; if the host does not expose the exact model, record that
fact. If the effective family is unknown, or an attempt cannot start or return,
mark that slot `unavailable` with the concrete reason. Do not fill an
unavailable slot with a second reviewer from the available family and do not
retry it through a new mechanism.

One returned review is sufficient when the other family is unavailable. If
neither review returns, have the retained writer checkpoint both unavailable
outcomes, then stop: the step has no independent review and cannot be
completed. Do not loop in the same run. A later run may freeze a new packet
only after provider authorization or availability has actually changed.

## Reconcile findings after isolation

Wait until both provider slots are terminal (`returned` or `unavailable`),
then reconcile the results. Preserve a source id for every finding, such as
`claude-B1` or `gpt-B1`; agreement between reviewers is useful context but is
not proof.

Every reviewer-labeled blocker must have one explicit disposition before the
step completes:

- `fixed` — cite the correcting commit and the check or artifact that proves
  the correction;
- `disproven` — cite contract, code, or test evidence showing why it is not a
  defect; or
- `operator decision` — use only for a genuine behavior or scope fork, and
  record the operator's answer plus the commit or evidence that applies it.

An unanswered operator decision is still open. Handle should-fixes under the
ordinary review triage. Handle subjective taste findings with the existing
fix-or-waive-with-reason rule; model votes do not replace that judgment.

The correcting checkpoint cannot contain its own commit hash. Record the
finding, disposition, and command/artifact evidence in that checkpoint; after
it returns, write its now-known hash into the ledger during finalization before
`sync-and-commit`. Do not use a placeholder as final evidence.

## Re-review only when evidence requires it

Do not run a routine second review. Re-review when a blocker resolution cannot
be proved objectively, materially changes the reviewed contract or approach,
or resolves conflicting reviewer concerns through a material decision.

Prefer a targeted return to the reviewer that raised the blocker, using a new
frozen packet for the correcting `HEAD`. Use both provider families only when
the correction changes the whole step's decision-relevant surface. Record why
re-review was or was not required and its result. Do not start an automatic
review loop; if a required re-review still leaves a genuine blocker, resolve
or surface that blocker normally.

## Keep a durable append-only resume ledger

The retained writer records a durable append-only review sequence in the
milestone implementation log. Each review attempt set gets a monotonically
numbered, uniquely headed entry such as `Step 1 review-pass-001`; its exact
ledger location is the qualified implementation-log path plus that unique
heading anchor. Keep these locations concrete—for example,
`specs/payments/payments-retry-impl.md#step-1-review-pass-002`—rather than
passing the implementation-log path or a general Step 1 section alone.

Each `review-pass` entry records:

- its pass id, exact ledger location, predecessor sequence entry, and status
  (`open`, `closed`, or `superseded`);
- frozen packet base and review commit SHAs;
- canonical tracker path, owning project, concrete project path, artifact stem,
  active semantic slug, claim checkpoint, and semantic branch root;
- Claude slot: `pending`, `returned`, or `unavailable`, plus effective identity
  or reason;
- GPT slot: the same fields;
- each finding id, severity, disposition, and evidence;
- taste triage;
- re-review trigger, packet, reviewer, and result, or `not required` with a
  reason; and
- `final-sync`: `pending` until review closure, then `ready` immediately before
  invoking `sync-and-commit`.

The sequence is append-only at the entry level. A writer may fill the current
open entry's pending outcomes and close it, but must never delete, rename, or
overwrite an earlier entry or its frozen packet. A targeted conditional
re-review stays inside its current `review-pass`; it is not a replacement full
pass.

When contract-change recovery repeats Step 1, append a uniquely headed
`Step 1 contract-rework-NNN` entry after the prior review pass and before any
new review pass. Record the triggering marker/decision, the exact prior-pass
location, rework base and checkpoint SHAs, changed contract clauses, corrected
Phase 4 baseline evidence, and status. Close that entry only when the rework evidence is
durable, then append a new `Step 1 review-pass-NNN` with fresh provider slots
and a fresh frozen packet. The canonical shape is
`review-pass-001 -> contract-rework-001 -> review-pass-002`. Never reuse the
old provider outcomes or edit away the old packet. If another rework is needed,
continue the monotonic sequence in the same way.

Initialize both provider slots as `pending` before the review-ready handoff.
On resume, trust a returned review only when the ledger ties it to the same
frozen packet. Complete missing attempts; do not repeat a recorded successful
attempt. If the branch moved before the provider attempts became terminal,
mark the current pass `superseded` with the reason, append the next numbered
pass with a new frozen packet, and do not combine reviews from different heads
as the required pair. Likewise, a later run after both providers were
unavailable appends a new pass only when availability or authorization actually
changed; it never overwrites the failed pass.

Resume from the final sequence entry, not from a broad Step heading. An open
`contract-rework` entry resumes contract rework; a closed `contract-rework`
without its successor appends the next `review-pass`; an open `review-pass`
resumes only that pass. A closed earlier pass never satisfies a later pass.
Verify every predecessor link and preserve all prior packets before changing
the current entry.

The step is review-complete only when both slots are terminal, at least one
review returned, every blocker and taste finding is dispositioned, every
required re-review is recorded complete, and the same tracker row remains
`active` under its frozen semantic identity. The retained writer then changes
`final-sync` to `ready` and invokes `sync-and-commit`, which commits that state.
On resume, an in-flight step is complete only when `ready` is present in the
clean branch `HEAD`; otherwise resume finalization. Pass the step number and
exact current review-pass location explicitly to `sync-and-commit`; that skill
validates the named pass and its preserved predecessor chain, and fails closed
on this ledger only for `ship-milestone` Steps 0, 1, and 2.
