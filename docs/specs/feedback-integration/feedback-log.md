# Feedback Log

Lifecycle: draft
Role: log
Project: agent-playbook-suite
Updated: 2026-06-08

## Purpose

This log records feedback considered under the Feedback Integration Process. Use it to keep a visible trail from raw feedback to routing decision, changed artifacts, validation, and remaining follow-up.

Related process: [Feedback Integration Process](process.md)

## Intake Template

```markdown
### <YYYY-MM-DD> — <short label>

- Source:
- Feedback:
- Example or evidence:
- Workflow moment:
- Owning skill or artifact:
- Adjacent skills or docs:
- Existing skill vs new skill decision:
- Intent preservation check:
- Planned change:
- Validation required:
- Status:
- Follow-up:
```

## Entries

### 2026-06-08 — Relax test/check gate language

- Source: operator feedback on previous commit language.
- Feedback: test and check requirements were too strong and overbuilt for ordinary feedback and use cases.
- Example or evidence: wording such as required explicit checks, full product suite, full quality gate, and fail-closed language made optional or risk-selected gates read as mandatory.
- Workflow moment: cross-skill quality model, milestone execution, sync/commit, ship-milestone review, and simplify verification.
- Owning skill or artifact: shared quality model plus `create-milestones`, `ship-milestone`, `sync-and-commit`, and `simplify` skill payloads.
- Adjacent skills or docs: `project-foundation` starter test-strategy wording and public `overview.md` simplification wording.
- Existing skill vs new skill decision: patch existing skills and shared references; no new skill needed because this changes gate selection policy inside existing workflows.
- Intent preservation check: compatible with existing intent. The change preserves safety rules against weakening tests, hidden-test leakage, unsafe git behavior, and docs drift, while reducing unnecessary ceremony for Lite and ordinary work.
- Planned change: replace blanket required/full-gate language with selected/configured gate language; make deep checks explicitly risk-driven.
- Validation required: `docs index .`, `docs check . --stale 14`, whitespace diff check, JSON manifest parsing, workflow payload and quality-model consistency checks; site build if Bundler is available.
- Status: implemented in working tree on `v1.7.0-feedback-use-cases`; docs validation and workflow-content checks passed; site build not run because `bundle` is unavailable.
- Follow-up: review final wording before committing and decide whether release/version metadata needs to move from plugin `0.3.2` toward the intended 1.7.0 line.

### 2026-06-08 — Add feedback integration process and internal docs layout

- Source: operator request to start a feedback process and better organize internal specs.
- Feedback: create a process for integrating new feedback into appropriate skills or deciding when to create a new skill; create `docs/` and `docs/specs/` to help the repository manage itself.
- Example or evidence: need to check which skill feedback relates to, where it should go, whether changes fundamentally modify skill intent, and what additional checks should be covered.
- Workflow moment: suite-maintenance process, not one runtime skill.
- Owning skill or artifact: root docs tree.
- Adjacent skills or docs: `docs` skill guidance for managed docs; future changes may affect public docs or skill payloads depending on feedback routing.
- Existing skill vs new skill decision: create a spec doc, not a runtime skill. This is a maintainer process for changing the suite.
- Intent preservation check: compatible with repository maintenance intent; does not change runtime skill behavior by itself.
- Planned change: create `docs/specs/feedback-integration-process.md`, move it under the new internal docs layout, and add `docs/README.md` explaining placement rules.
- Validation required: `docs index .` and `docs check . --stale 14`.
- Status: implemented in working tree; docs validation passed.
- Follow-up: keep this log updated for each new feedback item and consider adding links from public maintainer docs if the process becomes permanent.

### 2026-06-08 — Decision recording can be lost after interruption

- Source: operator feedback.
- Feedback: decisions are not always recorded properly. If something interrupts during the documentation process, the agent can lose focus and forget to write down an important decision.
- Example or evidence: pending discussion with operator.
- Workflow moment: documentation and decision capture during a workflow run, especially when interrupted or resumed.
- Owning skill or artifact: pending discussion.
- Adjacent skills or docs: pending discussion.
- Existing skill vs new skill decision: pending discussion.
- Intent preservation check: pending discussion.
- Planned change: none yet; intake and use-case discovery only.
- Validation required: update this log, then run `docs index .` and `docs check . --stale 14`.
- Status: logged for now; discovery paused; do not implement in process spec or skills yet.
- Follow-up: investigate when this happens, what interrupts the agent, what decision types are lost, and what evidence should be recorded before proposing changes.

### 2026-06-08 — Instructions and artifact intent can get co-mingled

- Source: operator feedback during feedback-process discussion.
- Feedback: agent-facing instructions and artifact intent can get co-mingled. When the operator tells the agent how to write or avoid something, the agent may copy that instruction verbatim into generated code, prompts, or planning docs where it does not make sense for the future reader or runtime consumer.
- Example or evidence: observed in this conversation. The operator asked to update the feedback process, and the process text initially included interruption-specific examples from the current feedback item instead of general discovery guidance. A related pattern: an instruction such as "make sure that you don't include local references" can turn into generated prompt text such as "not including local references". The generated prompt consumer would not know what "local references" means, so the instruction leaks context instead of becoming a useful artifact requirement.
- Diagnosis: the failure was overfitting the durable docs output to the current example. The agent did not sufficiently generalize the operator's instruction into a reusable process rule.
- Guardrail direction: use a pre-write translation check backed by a short post-write review. The check should separate operator instruction, artifact intent, and generalized rule before writing durable artifact text. Apply it mainly where instruction leakage is most likely, rather than adding ceremony to every suite-maintenance edit.
- Guardrail scope: prompt-writing and implementation/planning docs.
- Review risk: the bad output is usually not caught during review because the wording can look superficially reasonable even when it carries conversation-local context into a durable artifact.
- Downstream harm: confuses future agents, generates worse prompts, and can create incorrect docs or code behavior.
- Workflow moment: prompt writing, planning-doc writing, code/document generation, and any workflow step where operator instructions must be translated into durable artifact content.
- Owning skill or artifact: pending discussion.
- Adjacent skills or docs: pending discussion.
- Existing skill vs new skill decision: pending discussion.
- Intent preservation check: pending discussion.
- Planned change: none yet; intake and use-case discovery only.
- Validation required: update this log, then run `docs index .` and `docs check . --stale 14`.
- Status: logged for now; discovery paused; do not implement in process spec or skills yet.
- Follow-up: investigate where instruction leakage happens most often, which generated artifacts are affected, how to distinguish temporary authoring instructions from durable artifact text, and what review/checkpoint would catch it.

### 2026-06-08 — Tests are overspecified and correctness checking is too complex

- Source: operator feedback.
- Feedback: tests are overspecified, and the complexity around checking correctness is too high.
- Example or evidence: a project used a byte-exact golden test when that level of precision was not necessary. The golden made later expansion harder and created additional issues as the project evolved.
- Diagnosis: overspecified tests can overconstrain implementation choices, which pushes complexity into the resulting system rather than simply checking correctness.
- Preferred direction: semantic behavior should be the first priority for correctness checks.
- Overfitting caution: do not turn the byte-exact golden example into rigid guidance for every skill. Keep any future change simple: avoid freezing incidental representation.
- Working rule: prefer the least constraining check that gives real confidence in the intended behavior.
- Workflow moment: test planning, quality gate selection, correctness verification, milestone acceptance, and review.
- Owning skill or artifact: pending discussion.
- Adjacent skills or docs: pending discussion.
- Existing skill vs new skill decision: pending discussion.
- Intent preservation check: pending discussion.
- Planned change: none yet; intake and use-case discovery only.
- Validation required: update this log, then run `docs index .` and `docs check . --stale 14`.
- Status: intake recorded; discussion in progress.
- Follow-up: distinguish this from the earlier test/check language relaxation, identify what is overspecified, when correctness-check complexity appears, and what a simpler adequate correctness model should look like.

### 2026-06-08 — Milestones need a head feature branch merge target

- Source: operator feedback.
- Feedback: make sure milestone work has a head feature branch to merge back to, rather than merging milestone branches directly to `dev` or `main`, so milestones can always merge back into that feature branch.
- Example or evidence: the head feature branch should represent the project or feature effort, analogous to a topic directory such as `docs/specs/<feature-or-project>/`.
- Branch ownership direction: `project-foundation` should define the head feature branch convention for the project/feature effort: name, base branch, and milestone merge target. `ship-milestone` should verify that branch exists and use it as the base or merge target for milestone branch stacks.
- Missing branch behavior: ask before creating the head feature branch because branch creation affects repository workflow.
- Milestone merge behavior: merge milestone branches back into the head feature branch as each milestone completes, rather than waiting until the whole feature is ready.
- Review behavior: keep the existing human-review gate. The agent should prepare the completed milestone branch for review and identify the head feature branch as the merge target, but should not auto-merge.
- Workflow moment: branch planning, milestone branch-stack creation, milestone completion, and integration.
- Owning skill or artifact: pending discussion.
- Adjacent skills or docs: pending discussion.
- Existing skill vs new skill decision: pending discussion.
- Intent preservation check: pending discussion.
- Planned change: none yet; intake and use-case discovery only.
- Validation required: update this log, then run `docs index .` and `docs check . --stale 14`.
- Status: logged for now; discovery paused; do not implement in process spec or skills yet.
- Follow-up: investigate how the head feature branch should be named, who creates it, where milestone branches branch from and merge back, and how this relates to `ship-milestone`, `sync-and-commit`, and public branch-safety guidance.

### 2026-06-08 — Risk levels need clearer reasoning

- Source: operator feedback.
- Feedback: risk levels should be more explicit. High and Standard should be well thought-out and reasoned, including the situations where each level is used.
- Example or evidence: agents choose High too often, and they do not ask for the risk-level choice when initially creating the milestone or project plan in foundation work.
- Diagnosis: risk classification is currently too implicit and conservative. The workflow needs an earlier operator-facing choice and better reasoning around Standard vs High.
- Preferred direction: default to Standard unless there is an explicit reason for High. If the agent believes High applies, it should query the operator and explain the reason.
- High-risk trigger examples: security, auth, billing, data loss, destructive migrations, public API breakage, and production incident response.
- Lite vs Standard direction: reserve Lite for docs, internal/admin work, low-blast-radius refactors, and other changes with low impact. Use Standard as the default for ordinary product or workflow changes.
- Workflow moment: project foundation risk strategy, milestone creation, quality-gate selection, review, and handoff.
- Owning skill or artifact: pending discussion.
- Adjacent skills or docs: pending discussion.
- Existing skill vs new skill decision: pending discussion.
- Intent preservation check: pending discussion.
- Planned change: none yet; intake and use-case discovery only.
- Validation required: update this log, then run `docs index .` and `docs check . --stale 14`.
- Status: intake recorded; discussion in progress.
- Follow-up: identify what criteria distinguish Lite, Standard, and High; clarify when a milestone should be promoted or demoted; determine how much reasoning must be recorded without adding too much ceremony.
