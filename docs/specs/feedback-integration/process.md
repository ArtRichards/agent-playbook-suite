# Feedback Integration Process

Lifecycle: draft
Role: spec
Project: agent-playbook-suite
Updated: 2026-06-08

## Purpose

This spec defines how feedback, dogfooding notes, operator requests, and new use cases are evaluated before changing Agent Playbook Suite. The goal is to route feedback to the right skill, preserve each skill's intent, and make deliberate decisions about when to patch an existing skill versus create a new one.

## Inputs

Each feedback item should be captured with enough context to make routing possible:

- source: operator feedback, user issue, dogfooding note, failed run, review finding, marketplace install problem, docs-cli behavior, or external use case;
- concrete example: command, prompt, transcript excerpt, file path, failure mode, or desired workflow;
- affected host: Codex, Claude Code, both, or host-independent;
- affected artifact: public docs, marketplace metadata, plugin manifest, skill `SKILL.md`, skill README, skill references, shared quality model, generated site, or docs-cli guidance;
- expected outcome: clearer instruction, safer behavior, less friction, new capability, bug fix, or packaging change;
- risk if wrong: confusion, broken install, incorrect code edits, unsafe git behavior, over-constrained workflow, under-tested workflow, or scope drift.

## Intake And Discovery Workflow

Use a discovery-first process for each feedback item. Do not change skill
instructions, public docs, package metadata, or shared policy until the use case
is understood well enough to route.

1. Record the raw feedback in `feedback-log.md` before analyzing or proposing a
   fix.
2. Mark the item as discovery in progress.
3. Ask one investigation question at a time. Wait for the operator's answer
   before asking the next question.
4. Record each answer in the feedback-log entry before continuing.
5. Continue discovery until the log captures:
   - when the issue happens;
   - what event, condition, or workflow context exposes it;
   - which workflow phase, skill, or artifact is involved;
   - what happened, what should have happened, and why the difference matters;
   - what a good behavior or outcome would look like;
   - whether the issue is isolated, recurring, or systemic;
   - what evidence would show the feedback has been addressed.
6. Only after discovery is complete, propose routing, ownership, and possible
   changes.
7. Wait for operator confirmation before editing any runtime skill behavior.

Good discovery questions are concrete and narrow. Prefer questions about a real
run, transcript, command, file, artifact, failure mode, confusing wording, or
expected behavior over abstract design questions.

## Routing Questions

Start every feedback pass by answering these questions in order:

1. Which workflow moment does this feedback concern?

   - Project foundation and front-half planning.
   - Milestone creation, phase advancement, or milestone archival.
   - End-to-end autonomous milestone shipping.
   - Verification, docs sync, commit, or push behavior.
   - Post-implementation simplification.
   - Docs-cli usage and docs-tree mechanics.
   - Marketplace install, publishing, or packaging.
   - Public explanation in `README.md`, `overview.md`, `blog-post.md`, or the site.

2. Which existing skill owns that moment?

   - `project-foundation`: project charter, scope, architecture, milestone plan, risk strategy, Definition of Ready, root agent context.
   - `create-milestones`: milestone docs, test matrices, phase-by-phase interactive TDD, milestone completion and archive.
   - `ship-milestone`: conductor flow, branch stack, fresh sub-agent prompts, same-instance audit, fresh-eyes review, end-to-end orchestration.
   - `sync-and-commit`: verification, docs sync, git safety, commits, pushes, final reports.
   - `simplify`: behavior-preserving cleanup after implementation is already green.
   - `docs`: docs-cli commands, metadata convention, generated index, archive, migration, quality artifacts.
   - `_shared`: cross-skill policy that multiple workflow skills must apply consistently.

3. Is the feedback about skill behavior, public documentation, package metadata, or docs-cli itself?

   - Skill behavior belongs in the relevant skill payload under `plugins/agent-playbook-suite/skills/`.
   - Shared behavior belongs in `_shared` only when at least two workflow skills must consume the same policy.
   - Public explanation belongs in root docs and site files.
   - Marketplace or installation changes belong in `.agents/`, `.claude-plugin/`, plugin manifests, `README.md`, and `marketplace-publishing-guide.md`.
   - Runtime docs-cli behavior should be checked against the `docs-cli` source before changing suite guidance.

## Existing Skill vs New Skill

Patch an existing skill when the feedback changes how that skill performs its current job, clarifies a decision it already owns, removes friction in its existing workflow, or fixes a prompt/checklist that causes bad behavior.

Create a new skill only when the feedback introduces a repeatable workflow with its own trigger, inputs, outputs, stop conditions, and lifecycle that does not fit cleanly inside an existing skill. A new skill should have a distinct operator intent. It should not exist merely because a section grew long or because one user asked for a local variation.

Prefer a shared reference over a new skill when the change is a policy or checklist used by multiple skills, but only if sharing reduces drift. Do not move material into `_shared` when one skill is the only real owner.

## Intent Preservation Check

Before modifying a skill, state the skill's current intent in one paragraph and compare the proposed change against it.

The change is compatible when it makes the skill better at its existing purpose, clarifies ambiguous instructions, adds a guardrail for a known failure mode, or narrows over-broad language without changing ownership.

The change may modify intent when it adds a new phase, changes the skill's trigger, moves responsibility between skills, changes autonomy level, changes when operator approval is needed, changes git or docs safety behavior, or changes what evidence is required to declare work complete.

If a change modifies intent, treat it as a design decision: update public docs if adoption expectations change, update related skills so handoffs remain consistent, and record the rationale in the relevant guide or spec.

## Change Design Checklist

For each accepted feedback item:

- confirm discovery is complete enough to route;
- identify the owning skill or artifact;
- identify adjacent skills affected by handoff, terminology, or validation changes;
- decide whether the change is local, shared, public-facing, or marketplace-facing;
- check whether the change should alter install docs, publishing docs, or plugin manifests;
- check whether Codex and Claude Code need different wording;
- check whether examples, prompts, READMEs, and reference files need matching updates;
- check whether the docs root excludes the edited path from `docs check`;
- preserve the no-hand-edit rules for metadata, `INDEX.md`, archive paths, and generated docs content;
- avoid adding stricter gates unless the risk model justifies them;
- avoid relaxing safety rules that prevent hidden-test leakage, test weakening, git-hook bypass, unsafe pushes, or archive/index drift.

## Quality And Validation Checklist

Use the smallest validation set that covers the changed surface:

- For root docs: run `docs index .` and `docs check . --stale 14`.
- For marketplace metadata: parse changed JSON manifests with `python3 -m json.tool`.
- For skill payload shape: confirm the expected skill directories and `SKILL.md` files still exist.
- For shared quality-model changes: confirm every consuming skill still references the shared terms it depends on.
- For conductor or sub-agent prompt changes: check `ship-milestone` prompts, consistency audit, and `sync-and-commit` handoff language together.
- For docs-cli guidance: verify against the installed `docs --version` and, when behavior is uncertain, the public `docs-cli` source.
- For site-facing changes: build the site when local Ruby/Bundler dependencies are available; otherwise record that the site build was not run and why.

## Review Questions

Before closing a feedback integration pass, answer:

- Was the raw feedback recorded before any implementation change?
- Were investigation questions asked one at a time and were answers captured in
  the log?
- Does this make one skill responsible for a job another skill already owns?
- Does this add process weight to Lite or ordinary work without a clear risk reason?
- Does this weaken a safety invariant, or only relax unnecessary ceremony?
- Will a fresh agent with no conversation history know what to do from disk alone?
- Are hidden/private test cases still protected?
- Are docs-cli-managed files still mechanically valid?
- Are public install and publishing instructions still accurate?
- Does the marketplace payload still match the public story?

## Completion Criteria

A feedback integration change is complete when the owning artifact is updated, adjacent handoffs are consistent, public docs are updated if user expectations changed, required docs and metadata checks pass, and any skipped validation is explicitly reported with the reason.
