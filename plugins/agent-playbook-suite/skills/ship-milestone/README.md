# ship-milestone

An agent workflow skill that claims one semantic milestone from the canonical
`milestone-plan.md`, runs all ten risk-aware TDD phases, and safely closes the
milestone by archiving exactly its verified artifact set.

A lightweight conductor spawns fresh high-capability sub-agents for milestone
claim, creation (if needed), per-step planning and implementation, isolated
fresh-eyes reviews, and simplification/closeout. It commits each step to a
semantic branch stack and finishes with an explicit docs-cli archive plan.
Each sub-agent starts with a clean context, builds understanding from artifacts
on disk, and returns a structured report; the conductor's job is triage, not
implementation. For GPT-family agents, use Codex `gpt-5.6-sol` with `xhigh`
reasoning when available. For Claude-family agents, use the Claude Code `opus`
alias with `xhigh`; the alias tracks the newest supported Opus model. Record
any host or account model substitution instead of claiming the requested model
ran.

## Step model

The conductor reads tracker-owned identity, order, state, dependencies, and
eligibility. For `next milestone`, it selects the lexicographically first
eligible slug at the lowest eligible `Order`, then a retained writer changes
that row from `planned` to `active` before the first branch is created.
Reordering never renames files or branches.

The conductor walks the claimed milestone through four steps, each on its own
`<project-branch-prefix><slug>/...` branch. Project slugs are repository-globally
unique; the branch prefix is `<project>/` in repositories with multiple suite
projects and empty only for one project-wide namespace:

1. `<project-branch-prefix><slug>/milestone-setup` — milestone creation agent (only if the milestone's task plan, implementation log, or test matrix does not yet exist) → isolated fresh-eyes review gate.
2. `<project-branch-prefix><slug>/phases-1-4` — planning agent → implementation agent
   (contract, visible RED product tests or selected explicit non-product
   checks, or an adequate GREEN baseline for a pure refactor;
   hidden/generalization categories, test matrix) → isolated
   fresh-eyes review gate and risk-aware RED checkpoint.
3. `<project-branch-prefix><slug>/phases-5-10` — planning agent → implementation agent (implement, integrate, quality) → isolated fresh-eyes review gate.
4. `<project-branch-prefix><slug>/simplify` — retained simplify-and-close writer
   ([`/simplify`](https://github.com/ArtRichards/simplify) skill), a conditional
   single-reviewer-or-operator gate only for High-risk code changes, a clean
   pre-archive checkpoint, explicit archive preview/apply, tracker completion, then
   [`sync-and-commit`](https://github.com/ArtRichards/sync-and-commit).

Step 3 keeps semantic `<slug>` as tracker/branch identity and separately
resolves the path-free `<artifact-stem>` used by milestone filenames.
`<project-path>` is empty for a dedicated docs root or a trailing-slash
docs-root-relative prefix in a shared root. Before preview or apply, the
conductor freezes the literal `<project-path><artifact-stem>-*` scope and the
qualified primary, implementation-log, and test-matrix paths. It verifies that
the selected set is exactly those three artifacts, that planned destinations
are unique across the docs root because docs-cli flattens archive paths, and
that preview/apply use the identical scope. Before either archive command, the
writer commits a clean checkpoint whose tree is tied to any required High-risk
approval; routine Step 3 work does not run the Step 0–2 dual-provider protocol.
Long-lived planning docs, future or paused milestones, and cross-milestone
context remain active. After the move, the writer changes the same tracker row
from `active` to `complete`; sync verifies captured preview/apply JSON in the
normal path. If an interruption already moved the exact set, it verifies the
established branch diff, archive witnesses, rebased links, and clean docs state
from that checkpoint instead—without accepting later code changes, inventing
missing JSON, rerunning archive, or touching an archived milestone document.

At the end of each completed Step 0, 1, and 2, the conductor freezes one
decision-relevant evidence packet and independently attempts one Claude-family
and one GPT-family review. No returned report is shared with the other attempt
before both slots are terminal. If exactly one provider mechanism is
unavailable or is not already authorized in the active environment, the
conductor records that result and continues with exactly one reviewer from the
available provider; it does not add a second reviewer from the same provider.
If neither returns, the step stops without claiming independent review.
Installing the marketplace plugin supplies the workflow, not provider
accounts, credentials, or cross-provider access.

Every reviewer-labeled blocker must be fixed, disproven with contract, code, or
test evidence, or routed to the operator as a genuine product or scope
decision before the step completes. Re-review is conditional: target the
reviewer who raised a concern when objective evidence cannot close it, the
resolution materially changes the reviewed approach, or reviewer concerns
conflict. Repeat both isolated reviews only when a correction changes the
whole step's decision-relevant surface.

Within the phases-5-10 step, between its planning and implementation agents,
the conductor conditionally invokes
[`explore`](https://github.com/ArtRichards/agent-playbook-suite/tree/main/plugins/agent-playbook-suite/skills/explore)
only when a named technical decision still meets the suite's high-threshold
solution-uncertainty gate. The same skill can recover from a concretely
invalidated route. Every exploration result is checkpointed before planning or
contract-change recovery continues.

Each implementation agent runs the same-instance consistency / completeness /
accuracy audit (see
[`references/consistency-check.md`](references/consistency-check.md)) before
returning to the conductor. The independent review agents are read-only. The
conductor surfaces open questions to the operator and resumes the responsible
agent with answers.

## When to invoke

Use when the operator wants a milestone built end-to-end without per-phase
hand-holding. Invoke as `/ship-milestone <semantic-slug>` (for example,
`/ship-milestone session-storage`) or `/ship-milestone next milestone`.

## Install

Install this skill through the Agent Playbook Suite marketplace plugin. The
repository root `README.md` carries the Codex and Claude Code marketplace
commands. This directory is the portable skill payload for hosts that support
direct skill-directory installs.

## Dependencies

- [`docs-cli`](https://github.com/ArtRichards/docs-cli) 2.0 or newer — required
  for reciprocal relationships, body-link rebasing, explicit
  `--cascade-dry-run` / `--cascade-only` archival, and docs-tree validation.
  Install with `pip install --upgrade docs-cli`.
- Companion skills — required:
  - [`create-milestones`](https://github.com/ArtRichards/create-milestones) (process + 10-phase TDD structure).
  - [`sync-and-commit`](https://github.com/ArtRichards/sync-and-commit) (called at every step boundary).
  - [`simplify`](https://github.com/ArtRichards/simplify) (Step 3).
- Companion skill — conditional: `explore` (technical route selection and
  evidence-backed recovery only when the activation gate passes).
- Companion skill — recommended: [`project-foundation`](https://github.com/ArtRichards/project-foundation) (run once before the first milestone).
- `CLAUDE.md`, `AGENTS.md`, or equivalent project context at the project root documenting the docs tree location, build/test/quality commands, risk gates, commit convention, and branch convention. Sub-agents read this to bootstrap context.

## License

MIT — see [`LICENSE`](LICENSE).
