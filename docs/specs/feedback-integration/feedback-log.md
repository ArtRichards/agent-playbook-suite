# Feedback Log

Lifecycle: draft
Role: log
Project: agent-playbook-suite
Updated: 2026-06-12

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

### 2026-06-12 — Foundation should be more interactive and investigatory

- Source: operator feedback.
- Feedback: the foundations phase/skill could be more interactive and investigatory, to assist with both risk assessment and use case.
- Example or evidence: pending discovery. Related: the 2026-06-08 entry "Risk levels need clearer reasoning" already records that agents do not ask for the risk-level choice when initially creating the milestone or project plan in foundation work, and that High is chosen too often.
- Discovery notes:
  - 2026-06-12: operator confirmed foundation assigns risk levels without consulting the operator and over-emphasizes the risk. This corroborates the "Risk levels need clearer reasoning" entry; the fix for the risk half of this item should land together with that one.
  - 2026-06-12: operator confirmed foundation completely fails to probe how the product would actually be used, and diagnosed the root cause: it was never explicitly directed to query about use cases, rather than asking badly. Use-case exploration should therefore be a main separate part that runs once foundations are done — see the same-day entry "Create a use-case skill after foundation" — not an extra question inside foundation.
  - 2026-06-12: operator added an on-demand deep mode — if the user asks for it, foundation should launch a thorough investigation of the entire product's foundation, beyond the default investigatory grounding.
- Workflow moment: project foundation and front-half planning.
- Owning skill or artifact: `project-foundation` for the risk half and the handoff; the use-case half is delivered by the new `use-cases` skill (see same-day entry).
- Adjacent skills or docs: the `use-cases` skill (automatic handoff at foundation completion), foundation risk-strategy wording, `create-milestones` risk and gate selection, and the "Risk levels need clearer reasoning" entry whose criteria the risk half implements.
- Existing skill vs new skill decision: patch `project-foundation`; no new skill from this entry itself — the new-skill portion is owned by the "Create a use-case skill after foundation" entry.
- Intent preservation check: changes when operator input is requested (interactive risk consultation), which the process treats as an intent-level change — here it is deliberate and operator-directed. The handoff addition is part of the use-cases stage design decision.
- Planned change (proposed, awaiting operator confirmation): (1) `project-foundation` consults the operator on risk levels interactively — propose a level with reasoning, default Standard, require explicit operator approval for High, in line with the "Risk levels need clearer reasoning" direction; (2) foundation grounds its proposals investigatively (inspect the repo or product before proposing risk and scope rather than assuming), and when the operator asks for it, launches a thorough investigation of the entire product's foundation; (3) foundation hands off automatically to the `use-cases` skill at completion.
- Validation required: update this log, then run `docs touch <this log> --check` (stale window from `.docs.toml [check] stale_days`); confirm risk wording stays consistent between project-foundation, create-milestones, and the shared quality model.
- Status: implemented in working tree on `v1.7.0-feedback-use-cases` (2026-06-12). Foundation playbook gains "Ground every phase in investigation" (with the operator-requested deep-investigation mode) and the Phase 7 "Propose risk levels, never assign them" protocol (default Standard; High requires explicit operator approval); SKILL.md gains the matching invariant, investigation-first per-phase format, and the use-cases handoff.
- Follow-up: the broader "Risk levels need clearer reasoning" entry stays open for cross-skill criteria work (e.g. create-milestones' "choose the higher plausible level" wording).

### 2026-06-12 — Create a use-case skill after foundation

- Source: operator feedback.
- Feedback: create a "Use-Case" skill that runs after the foundations, so use cases can be explored which will guide the testing.
- Example or evidence: pending discovery. Related: the 2026-06-08 entry "Tests are overspecified and correctness checking is too complex" prefers semantic behavior as the first priority for correctness checks; explored use cases would supply that semantic grounding.
- Discovery notes:
  - 2026-06-12: operator clarified the stage's job — once the foundations are done, understand what the use cases are and how the user should use the product, and highlight these as primary goals.
  - 2026-06-12: operator added that when working with feedback, the workflow should also gather information about use cases — feedback discovery should include use-case probing, not only failure-mode probing.
  - 2026-06-12: operator clarified the exploration style — use-case exploration should be open-minded, and the agent should provide suggestions as well, not just blindly demand use cases from the operator. It should work together with the operator to understand how a user could really utilize the software or tool, with best UX in mind.
  - 2026-06-12: operator defined the artifact — a use-cases doc with primary use cases, mapped to test matrices. Tests should focus on these use cases primarily. This supplies the semantic anchor the "Tests are overspecified" entry asks for: use-case coverage outranks incidental-representation checks.
  - 2026-06-12: operator defined invocation — a separate skill that runs automatically after foundations complete, and that can also be invoked directly when working with feedback or when discussing use cases.
  - 2026-06-12: operator clarified the stage can be optional, but is strongly preferred. Milestone work should function without a use-cases doc, while the workflow nudges toward running the stage rather than silently skipping it.
- Workflow moment: between project foundation and milestone creation; test planning and quality-gate selection; revisited during feedback work.
- Owning skill or artifact: new `use-cases` skill in the suite bundle. Output artifact: a use-cases doc in the project's foundation docs directory listing primary use cases, highlighted as primary goals, with each mapped to test-matrix coverage.
- Adjacent skills or docs: `project-foundation` (automatic handoff at completion; role-mapping row for the use-cases doc), `create-milestones` (test matrices map tests to use cases; tests focus on them primarily), `_shared` quality model (use-case coverage as the first-priority semantic check), the feedback workflow (feedback discovery gathers use-case information and updates the doc), plus plugin manifest, marketplace metadata, and public README/overview which describe the suite's workflow stages.
- Existing skill vs new skill decision: new skill, operator-directed and process-justified — distinct operator intent (explore how users would really use the tool, best UX in mind), trigger (automatic after foundation; direct invocation during feedback or use-case discussion), inputs (foundation docs plus collaborative operator dialogue with agent-contributed suggestions), outputs (use-cases doc and matrix mappings), stop condition (primary use cases agreed and recorded).
- Intent preservation check: modifies the suite's public workflow story by adding a stage between foundation and milestones — a deliberate design decision. Public docs and adjacent skill handoffs must be updated together; no autonomy or git-safety changes.
- Planned change (proposed, awaiting operator confirmation): (1) create skill payload `plugins/agent-playbook-suite/skills/use-cases/` with SKILL.md and references for the collaborative exploration playbook and the use-cases doc template; (2) `project-foundation` hands off to it automatically at completion and adds the role-mapping row; (3) `create-milestones` test matrices gain use-case mapping, with tests focused primarily on use cases; (4) plugin manifest, both marketplace JSONs, and public README/overview updated to describe the new stage.
- Validation required: update this log, then run `docs touch <this log> --check` (stale window from `.docs.toml [check] stale_days`); parse changed JSON manifests with `python3 -m json.tool`; confirm skill payload shape; check handoff wording across project-foundation, use-cases, and create-milestones together.
- Status: implemented in working tree on `v1.7.0-feedback-use-cases` (2026-06-12). New `use-cases` skill payload created (SKILL.md plus use-cases-playbook reference with the `docs new spec use-cases` template); project-foundation hands off automatically; create-milestones reads `use-cases.md` at bootstrap and its test-matrix template gains a Use-case mapping section; Codex plugin longDescription, README, overview, and the generated-context template updated.
- Follow-up: review wording before commit; the plugin version question (0.3.2 vs the 1.7.0 line) remains open for the release step.

### 2026-06-12 — New milestones need appropriate placement and numbering

- Source: operator feedback.
- Feedback: changes to milestones — ensure that any new milestones are placed and numbered appropriately. Operator examples: `m4a5c`, `m4b`, `m6c`; starting with `m5`, then `m5a` and `m5b`, then `m5a1`, `m5a2`, `m5a2a`, and so on.
- Example or evidence: the examples describe a hierarchical identifier scheme that alternates letter and number suffixes, allowing new milestones to be inserted at any point in an existing plan at increasing depth (`m5` → `m5a` → `m5a1` → `m5a2a`) without renumbering existing milestones.
- Discovery notes:
  - 2026-06-12: operator clarified suffix semantics — a suffixed id primarily means the milestone is inserted into the sequence (e.g. `m5a` runs between `m5` and `m6`); the same scheme can also express a sub-milestone that decomposes its parent, as needed.
  - 2026-06-12: repo survey — milestone ids are `m<N>` or `m<N>-<topic>` throughout the payloads: milestone-playbook naming guidance, project-foundation role-mapping rows (`m<N>`, `m<N>-impl`, `m<N>-test-matrix`, `quality/m<N>-quality-log`), and ship-milestone branch slugs (e.g. `m4`). All current examples assume integer ids; hierarchical ids are filename- and branch-safe, so the change is convention wording rather than mechanics.
  - 2026-06-12: operator confirmed the current failure modes — when new work surfaces mid-plan, the agent either appends it at the end of the plan or tries to renumber existing milestones. Renumbering is especially harmful because ids appear in doc filenames, `Related:` pairing, branch names, and archived status tables.
- Workflow moment: milestone creation and insertion into an existing milestone plan.
- Owning skill or artifact: `create-milestones` owns id selection at milestone creation; its playbook gains the insertion convention. `project-foundation` documents the id grammar in the milestone plan and role-mapping so initial plans set the expectation.
- Adjacent skills or docs: `ship-milestone` ("next milestone" selection must order suffixed ids correctly; branch slugs already accept them); docs-tree naming of milestone/impl-log/test-matrix/quality-log companions follows the id automatically.
- Existing skill vs new skill decision: patch existing skills; no new skill. Id selection is a decision `create-milestones` already owns; this clarifies it and adds a guardrail.
- Intent preservation check: compatible. Clarifies an existing decision and adds a never-renumber guardrail for a known failure mode; no autonomy, approval, gate, or ownership changes.
- Planned change (proposed, awaiting operator confirmation): (1) `create-milestones` milestone-playbook — id convention: planned sequence uses `m<N>`; work inserted between existing milestones appends an alternating letter/number suffix to the predecessor id (`m5a` between `m5` and `m6`, `m5a1` between `m5a` and `m5b`, deeper as needed, e.g. `m5a2a`); the same scheme may denote sub-milestones decomposing a parent; never renumber existing milestones; confirm the proposed id with the operator when inserting. (2) `project-foundation` role-mapping and milestone-plan wording — id grammar covers suffixed ids. (3) `ship-milestone` — define "next milestone" ordering to respect suffix depth (numeric segments compare numerically).
- Validation required: update this log, then run `docs touch <this log> --check` (stale window from `.docs.toml [check] stale_days`); for skill payload edits, confirm skill directories and `SKILL.md` files remain intact and id wording stays consistent across the three skills.
- Status: implemented in working tree on `v1.7.0-feedback-use-cases` (2026-06-12). create-milestones playbook gains the "Milestone ids: insert, never renumber" convention with operator confirmation for inserted ids; project-foundation role-mapping documents the suffixed id grammar; ship-milestone's next-milestone resolution and branch-slug note respect suffix ordering.
- Follow-up: none beyond the release step.

### 2026-06-12 — Keep a feedback log

- Source: operator feedback.
- Feedback: keep a feedback log. Received verbatim as "feedback: keep a feedback log"; scope not yet specified.
- Example or evidence: this repository already keeps `docs/specs/feedback-integration/feedback-log.md` under the Feedback Integration Process. The feedback arrived as a bare directive, so it is unclear whether it confirms the existing maintainer practice or asks the suite's runtime skills to establish and maintain a feedback log inside projects they manage.
- Discovery notes:
  - 2026-06-12: operator clarified scope — in suite-managed projects, feedback often gets integrated into the follow-on notes of the last milestone instead of a dedicated place. Feedback, ideas, and other such items should instead live in their own log, using the intake format this repository uses for feedback (the intake template in this log).
  - 2026-06-12: repo survey — the suite's skill payloads currently route such items to milestone follow-up fields: `create-milestones` milestone-playbook has "skipped deep gates and follow-up" and "Adequacy gaps or follow-ups" sections, and the shared agentic-quality-model logs deferred deep gates as follow-up. No skill currently creates or maintains a dedicated feedback/ideas log in managed projects.
  - 2026-06-12: operator decided there should be two project-level logs: an engineering follow-up log (adequacy gaps, skipped deep gates, deferred test work, and similar milestone-bound engineering items) and a feedback/ideas/scope log (operator feedback, new ideas, and scope thoughts that surface during work).
  - 2026-06-12: the log is the single home for open items; milestone docs reference it rather than holding follow-ups inline. The goal is to not lose this information when it is not acted upon. Once an item is acted upon, it should be removed from the log and rehomed to where the action happens (for example, into a milestone).
  - 2026-06-12: log references in milestone docs are transient — a milestone's unfinished entries can mention the log, but once an item is incorporated there is no need to mention the log anymore.
  - 2026-06-12: `project-foundation` should create both logs at bootstrap as part of the docs tree.
  - 2026-06-12: the engineering follow-up log should use a tighter entry shape (for example item, source milestone, risk, owner, status); the intake template is overkill there. The intake-style template applies only to the feedback/ideas/scope log.
  - 2026-06-12: operator confirmed the routing proposal, including log names `followup-log.md` and `feedback-log.md` placed alongside the other foundation docs; implementation deferred while the feedback pass continues.
- Workflow moment: milestone completion and follow-up capture; feedback and idea intake at any point during suite-managed project work.
- Owning skill or artifact: `project-foundation` creates both logs at bootstrap with their entry templates embedded in each log doc (as this repository's feedback log embeds its intake template). `create-milestones` routes engineering follow-ups to the follow-up log instead of holding them inline, references open items from milestone docs, and rehomes entries when a milestone incorporates them.
- Adjacent skills or docs: `ship-milestone` consistency-check and agent prompts ("logged as follow-up" wording), `_shared` agentic-quality-model deferred-gate language, `project-foundation` role-mapping and CLAUDE.md template, and possibly public `overview.md` if adoption expectations change.
- Existing skill vs new skill decision: patch existing skills; no new skill. This changes where existing workflows record follow-ups and feedback, which fits cleanly inside current skill ownership. Entry formats live in the generated log docs themselves rather than `_shared`, so writing skills follow the template found in the log file.
- Intent preservation check: compatible for `project-foundation` (two more bootstrap docs) and `create-milestones` (better at not losing follow-ups; changes where completion evidence is recorded, not what is required). No autonomy, approval, git-safety, or gate-strength changes.
- Planned change (proposed, awaiting operator confirmation): (1) `project-foundation` bootstraps two `log` docs — an engineering follow-up log with a tight entry shape and a feedback/ideas/scope log with an intake template — and mentions them in role-mapping and the CLAUDE.md template; (2) `create-milestones` playbook/tdd-phases record deferred gates and adequacy gaps as follow-up-log entries, reference open items transiently from milestone docs, and sweep/rehome entries at milestone planning and completion; (3) `ship-milestone` and `_shared` quality-model follow-up language points at the project follow-up log.
- Validation required: update this log, then run `docs touch <this log> --check` (stale window from `.docs.toml [check] stale_days`); for skill payload edits, confirm skill directories and `SKILL.md` files remain intact and consuming skills still reference shared terms.
- Status: implemented in working tree on `v1.7.0-feedback-use-cases` (2026-06-12). project-foundation now bootstraps `followup-log.md` (tight entry shape) and `feedback-log.md` (intake-style entries) with templates embedded in each doc, plus role-mapping/SKILL.md/generated-context rows; create-milestones sweeps the logs at milestone planning and completion, routes deferrals to them, and keeps milestone-doc references transient; ship-milestone's consistency check and the shared quality model point follow-up language at `followup-log.md`.
- Follow-up: none beyond the release step.

### 2026-06-09 — Project docs layout should be topic-oriented

- Source: operator feedback.
- Feedback: the suite should maintain a docs or specs directory in an appropriate place for the given project. Internal specs that have companion artifacts should use `docs/specs/<topic>/`, keeping the main process or design doc, logs, examples, decisions, and validation notes together. `Role:` metadata should describe what each file is, while the topic directory groups related material for humans.
- Example or evidence: this feedback was given while discussing the existing `docs/specs/feedback-integration/` area. The operator interrupted an attempted implementation and clarified that this is feedback that should follow the intake process before changes are made.
- Discovery notes:
  - 2026-06-09: operator clarified that the feedback is about `project-foundation` choosing the right locations when bootstrapping a project.
  - 2026-06-09: when a repository already has a top-level `docs/` directory but no `.docs.toml`, `project-foundation` should inspect existing directories to determine the right placement. Prefer `docs/specs/<project-slug>/` when that fits the project layout.
  - 2026-06-09: when a repository does not already have a usable internal docs directory, prefer creating `docs/specs/<project-slug>/`. Before choosing any existing documentation directory, determine whether it is internal project documentation or public/customer-facing documentation, such as docs pages or guides intended for external users. Avoid placing internal specs inside public documentation surfaces.
  - 2026-06-09: if `docs/` already exists but is public/customer-facing documentation, `project-foundation` should choose a new internal-docs directory name instead of placing specs inside the public docs tree.
  - 2026-06-09: public/customer-facing documentation signals include a documentation builder, a tool that builds docs for customers, or a directory that holds built/generated customer docs. If `docs/` appears to be public/customer-facing, prefer `internal-docs/specs/<project-slug>/` as the internal specs location, but ask the operator before creating it.
  - 2026-06-09: `project-foundation` should also avoid placing internal specs in directories that look like site or publishing surfaces, such as `site/`, `website/`, `pages/`, or `public/`, even if they contain Markdown.
  - 2026-06-09: if `project-foundation` finds an existing `.docs.toml`, it should inspect that docs root before deciding whether to reuse it for internal project specs.
  - 2026-06-09: when an existing docs root is appropriate for internal specs, default to placing the project foundation under `<docs-root>/specs/<project-slug>/`.
  - 2026-06-09: operator confirmation should depend on repository maturity. In a large existing codebase, confirm before creating a new docs root or internal specs directory. In a relatively greenfield project, proceed with the preferred default when the layout is clear.
- Workflow moment: project foundation bootstrap and docs-root/layout selection.
- Owning skill or artifact: `project-foundation` (foundation-playbook Step 0 owns docs-root detection and selection).
- Adjacent skills or docs: `docs` skill guidance and `docs-cli` tree mechanics.
- Existing skill vs new skill decision: patch `project-foundation`; no new skill. Docs-root selection is a decision Step 0 already owns; the discovery notes refine it.
- Intent preservation check: compatible. Refines an existing decision with placement rules and public-docs-surface guardrails; the maturity-dependent confirmation matches the operator-approval behavior foundation already has.
- Planned change (proposed, awaiting operator confirmation): expand foundation-playbook Step 0 docs-root detection with the recorded rules — inspect existing directories before placing anything; prefer `docs/specs/<project-slug>/` (or `<docs-root>/specs/<project-slug>/` when an appropriate `.docs.toml` root exists); classify documentation directories as internal vs public/customer-facing before reuse (signals: docs builders, generated customer docs, `site/`, `website/`, `pages/`, `public/`) and never place internal specs in public surfaces; prefer `internal-docs/specs/<project-slug>/` when `docs/` is public, asking before creating it; confirm before creating a new root in a mature codebase, proceed on the clear default in greenfield repos.
- Validation required: update this log, then run `docs touch <this log> --check` (stale window from `.docs.toml [check] stale_days`).
- Status: implemented in working tree on `v1.7.0-feedback-use-cases` (2026-06-12, operator-approved). Foundation-playbook Step 0 now inspects before placing: internal vs public classification with the recorded signals, the `docs/specs/<project-slug>/` and `<docs-root>/specs/<project-slug>/` defaults, the `internal-docs/specs/` fallback with an ask, the site/publishing-surface exclusions, and maturity-scaled confirmation. The docs-toml-template and role-mapping needed no changes.
- Follow-up: none beyond the release step.

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
- Discovery notes:
  - 2026-06-12: the project-logs work (see "Keep a feedback log") gives decisions, follow-ups, and feedback durable single homes (`decision-log.md`, `followup-log.md`, `feedback-log.md`), which narrows this item to the capture moment — remembering to write during or after an interruption — rather than where records live.
- Workflow moment: documentation and decision capture during a workflow run, especially when interrupted or resumed.
- Owning skill or artifact: pending discussion.
- Adjacent skills or docs: pending discussion.
- Existing skill vs new skill decision: pending discussion.
- Intent preservation check: pending discussion.
- Planned change: none yet; intake and use-case discovery only.
- Validation required: update this log, then run `docs touch <this log> --check` (stale window from `.docs.toml [check] stale_days`).
- Status: logged for now; discovery paused; do not implement in process spec or skills yet.
- Follow-up: investigate when this happens, what interrupts the agent, what decision types are lost, and what evidence should be recorded before proposing changes.

### 2026-06-08 — Instructions and artifact intent can get co-mingled

- Source: operator feedback during feedback-process discussion.
- Feedback: agent-facing instructions and artifact intent can get co-mingled. When the operator tells the agent how to write or avoid something, the agent may copy that instruction verbatim into generated code, prompts, or planning docs where it does not make sense for the future reader or runtime consumer.
- Example or evidence: observed in this conversation. The operator asked to update the feedback process, and the process text initially included interruption-specific examples from the current feedback item instead of general discovery guidance. A related pattern: an instruction such as "make sure that you don't include local references" can turn into generated prompt text such as "not including local references". The generated prompt consumer would not know what "local references" means, so the instruction leaks context instead of becoming a useful artifact requirement.
- Diagnosis: the failure was overfitting the durable docs output to the current example. The agent did not sufficiently generalize the operator's instruction into a reusable process rule.
- Guardrail direction: use a pre-write translation check backed by a short post-write review. The check should separate operator instruction, artifact intent, and generalized rule before writing durable artifact text. Apply it mainly where instruction leakage is most likely, rather than adding ceremony to every suite-maintenance edit.
- Guardrail scope: prompt-writing and implementation/planning docs.
- Discovery notes:
  - 2026-06-12: the guardrail direction was exercised informally while implementing the 1.7.0 feedback pass — operator instructions were generalized into the new log templates and use-cases playbook rather than copied verbatim. The item stays open for a durable, written guardrail in the prompt-writing and planning-doc workflows.
- Review risk: the bad output is usually not caught during review because the wording can look superficially reasonable even when it carries conversation-local context into a durable artifact.
- Downstream harm: confuses future agents, generates worse prompts, and can create incorrect docs or code behavior.
- Workflow moment: prompt writing, planning-doc writing, code/document generation, and any workflow step where operator instructions must be translated into durable artifact content.
- Owning skill or artifact: pending discussion.
- Adjacent skills or docs: pending discussion.
- Existing skill vs new skill decision: pending discussion.
- Intent preservation check: pending discussion.
- Planned change: none yet; intake and use-case discovery only.
- Validation required: update this log, then run `docs touch <this log> --check` (stale window from `.docs.toml [check] stale_days`).
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
- Discovery notes:
  - 2026-06-12: the use-cases work (see "Create a use-case skill after foundation") supplies the semantic anchor this entry asks for — a use-cases doc with tests focused primarily on the use cases, and a use-case mapping in every test matrix. The remaining scope is codifying the working rule itself in test-writing guidance.
  - 2026-06-12: operator-requested research — popular Hacker News discussions illustrating the issue (points/comments from the HN Algolia API):
    - [Test-induced design damage](https://news.ycombinator.com/item?id=7666866) (DHH, 2014; 243 pts / 155 comments) — contorting design to satisfy testing demands pushes complexity into the system ("needless indirection and conceptual overhead"). Same complexity-push outcome as this entry's diagnosis, reached via testability demands rather than representation exactness.
    - [Why Most Unit Testing is Waste](https://news.ycombinator.com/item?id=7353767) (Coplien; 306/268 in 2014, re-surfaced at 282/280 in 2016 and 258/158 in 2017) — low-level tests that pin implementation rather than behavior cost more than the confidence they buy.
    - [Write tests. Not too many. Mostly integration.](https://news.ycombinator.com/item?id=15565875) (Kent C. Dodds; 362/331 in 2017, 331/152 in 2019) — right-sizing the test surface toward behavior users observe.
    - [Is TDD Dead?](https://news.ycombinator.com/item?id=24281195) (Fowler/Beck/DHH series, 2014; 445/476 on the 2020 repost, plus [TDD is dead. Long live testing.](https://news.ycombinator.com/item?id=7633254) at 351/161) — the debate's overtesting and test-induced-damage threads.
    - [Change-Detector Tests Considered Harmful](http://googletesting.blogspot.com/2015/01/testing-on-toilet-change-detector-tests.html) (Google Testing Blog, 2015) — negligible HN traction but the most precise statement of the failure mode: assertions that mirror the implementation and break on every change while catching no bugs.
  - 2026-06-12: follow-up research — related articulations of "semantic behavior first / least constraining check" (content verified against the sources):
    - [Test Desiderata](https://news.ycombinator.com/item?id=21295575) (Kent Beck, 2019; 61 pts) — the canonical form: tests should be *behavioral* ("sensitive to changes in the behavior of the code under test") and *structure-insensitive* ("tests should not change their result if the structure of the code changes").
    - [Metamorphic Testing](https://news.ycombinator.com/item?id=19817413) (Hillel Wayne, 2019; 54 pts) — constrain relations between outputs instead of exact outputs ("at no point do we need to check what `out` is"); useful when exact oracles are expensive. The shared quality model already names metamorphic checks; this is the explainer.
    - Property-based testing — [Hypothesis docs](https://news.ycombinator.com/item?id=45818562) (231 pts / 161 comments, 2025) and [The sad state of property-based testing libraries](https://news.ycombinator.com/item?id=40875559) (208/121, 2024) — assert properties that "should pass for all inputs in whatever range you describe" rather than exact outputs; the least-constraining idea generalized.
    - [Testing Without Mocks: A Pattern Language](https://news.ycombinator.com/item?id=34293631) (James Shore; 113/76 in 2023, reposts at 90/62 and 55/28) — interaction-based mock tests "lock in implementation, making refactoring difficult"; favors sociable, state-based tests that check outcomes "without any awareness of its implementation". Frozen interaction structure is the mock-world twin of the frozen-representation failure mode.
    - TDD: Where Did It All Go Wrong (Ian Cooper, 2017 talk) — widely cited for "test behaviors at the public API; the unit of isolation is the test, not the class"; video only, low HN traction, content not independently verified here.
- Workflow moment: test planning, quality gate selection, correctness verification, milestone acceptance, and review.
- Owning skill or artifact: `create-milestones` (tdd-phases Phase 2 test-writing guidance) and the `_shared` quality model.
- Adjacent skills or docs: `ship-milestone` fresh-eyes review (which judges whether tests pin the contract and should equally flag overconstraint) and `project-foundation` test-strategy wording.
- Existing skill vs new skill decision: patch existing skills; no new skill. This adjusts how existing test-planning guidance constrains checks.
- Intent preservation check: compatible. Adds a calibration rule for an existing activity; does not weaken the safety rules against test weakening or hidden-test leakage — it governs how new checks are chosen, not whether existing ones may be relaxed.
- Planned change (proposed, awaiting operator confirmation): add the working rule to tdd-phases Phase 2 and the shared quality model — prefer the least constraining check that gives real confidence in the intended behavior; check semantic behavior first; do not freeze incidental representation (e.g. byte-exact goldens where semantic comparison suffices); have the fresh-eyes review flag overconstrained tests alongside under-constrained ones.
- Validation required: update this log, then run `docs touch <this log> --check` (stale window from `.docs.toml [check] stale_days`).
- Status: implemented in working tree on `v1.7.0-feedback-use-cases` (2026-06-12, operator-approved). The shared quality model gains a "Check calibration" section; tdd-phases Phase 2 activities and exit criteria carry the least-constraining-check rule; ship-milestone's fresh-eyes Step 1 clause and the consistency check now flag overconstrained tests (frozen incidental representation) alongside under-constrained ones. Combined with the use-cases stage, tests anchor on semantic use-case behavior first.
- Follow-up: none beyond the release step.

### 2026-06-08 — Milestones need a head feature branch merge target

- Source: operator feedback.
- Feedback: make sure milestone work has a head feature branch to merge back to, rather than merging milestone branches directly to `dev` or `main`, so milestones can always merge back into that feature branch.
- Example or evidence: the head feature branch should represent the project or feature effort, analogous to a topic directory such as `docs/specs/<feature-or-project>/`.
- Branch ownership direction: `project-foundation` should define the head feature branch convention for the project/feature effort: name, base branch, and milestone merge target. `ship-milestone` should verify that branch exists and use it as the base or merge target for milestone branch stacks.
- Missing branch behavior: ask before creating the head feature branch because branch creation affects repository workflow.
- Milestone merge behavior: merge milestone branches back into the head feature branch as each milestone completes, rather than waiting until the whole feature is ready.
- Review behavior: keep the existing human-review gate. The agent should prepare the completed milestone branch for review and identify the head feature branch as the merge target, but should not auto-merge.
- Discovery notes:
  - 2026-06-12: implementation surfaces identified while editing adjacent material — `ship-milestone`'s branch table and pre-flight, the generated context template's Branch conventions section, and `sync-and-commit` push rules are where this would land once discovery resumes. No behavior changed.
- Workflow moment: branch planning, milestone branch-stack creation, milestone completion, and integration.
- Owning skill or artifact: pending discussion.
- Adjacent skills or docs: pending discussion.
- Existing skill vs new skill decision: pending discussion.
- Intent preservation check: pending discussion.
- Planned change: none yet; intake and use-case discovery only.
- Validation required: update this log, then run `docs touch <this log> --check` (stale window from `.docs.toml [check] stale_days`).
- Status: logged for now; discovery paused; do not implement in process spec or skills yet.
- Follow-up: investigate how the head feature branch should be named, who creates it, where milestone branches branch from and merge back, and how this relates to `ship-milestone`, `sync-and-commit`, and public branch-safety guidance.

### 2026-06-08 — Risk levels need clearer reasoning

- Source: operator feedback.
- Feedback: risk levels should be more explicit. High and Standard should be well thought-out and reasoned, including the situations where each level is used.
- Example or evidence: agents choose High too often, and they do not ask for the risk-level choice when initially creating the milestone or project plan in foundation work.
- Discovery notes:
  - 2026-06-12: while discussing foundation interactivity (see "Foundation should be more interactive and investigatory"), operator reiterated that foundation assigns risk levels without consulting the operator and over-emphasizes the risk. Foundation should ask for the risk-level choice interactively.
  - 2026-06-12: the foundation half is implemented — Phase 7 now proposes risk levels with reasoning, defaults to Standard, and requires explicit operator approval for High. The remainder of this entry covers milestone creation and the shared model.
- Diagnosis: risk classification is currently too implicit and conservative. The workflow needs an earlier operator-facing choice and better reasoning around Standard vs High.
- Preferred direction: default to Standard unless there is an explicit reason for High. If the agent believes High applies, it should query the operator and explain the reason.
- High-risk trigger examples: security, auth, billing, data loss, destructive migrations, public API breakage, and production incident response.
- Lite vs Standard direction: reserve Lite for docs, internal/admin work, low-blast-radius refactors, and other changes with low impact. Use Standard as the default for ordinary product or workflow changes.
- Workflow moment: project foundation risk strategy, milestone creation, quality-gate selection, review, and handoff.
- Owning skill or artifact: `_shared` quality model (level-selection rule) and `create-milestones` (Step 1 risk confirmation wording). Foundation half already done under "Foundation should be more interactive and investigatory".
- Adjacent skills or docs: `ship-milestone` (risk-gate decisions in review and the RED-baseline checkpoint consume the assigned level; wording should stay consistent).
- Existing skill vs new skill decision: patch existing skills and the shared reference; no new skill.
- Intent preservation check: compatible. Changes how a level is chosen (default Standard, reasoned, operator-confirmed for High), not what each level gates; High-risk safety behavior (RED-baseline pause, deeper gates) is unchanged once a level is assigned.
- Planned change (proposed, awaiting operator confirmation): (1) add a "Choosing a level" rule to the shared quality model — default Standard for ordinary product or workflow changes; Lite reserved for docs, internal/admin, and low-blast-radius work; High only with an explicit trigger (the model's High list), proposed with the reason and confirmed by the operator; (2) align `create-milestones` Step 1 — replace "if risk is ambiguous, choose the higher plausible level or pause" with propose-with-reasoning, default Standard, operator confirmation, High requiring explicit approval.
- Validation required: update this log, then run `docs touch <this log> --check` (stale window from `.docs.toml [check] stale_days`).
- Status: implemented in working tree on `v1.7.0-feedback-use-cases` (2026-06-12, operator-approved). The shared quality model gains a "Choosing a level" rule (default Standard; Lite reserved; High only with a stated trigger and explicit operator approval), and `create-milestones` Step 1's "choose the higher plausible level" wording is replaced with propose-with-reasoning per that rule. The foundation half landed earlier the same day.
- Follow-up: promotion/demotion of an in-flight milestone's level and how much reasoning to record remain open; revisit if dogfooding shows the need.
