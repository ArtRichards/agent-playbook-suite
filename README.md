# Agent Playbook Suite

Lifecycle: active
Role: guide
Project: agent-playbook-suite
Updated: 2026-06-12

Agent Playbook Suite is a single marketplace-distributed plugin for agent
workflow skills. It packages the project-planning, milestone-delivery,
verification, and cleanup skills that are used together for docs-backed
software delivery.

The suite plugin includes:

- `docs`
- `project-foundation`
- `use-cases`
- `create-milestones`
- `ship-milestone`
- `sync-and-commit`
- `simplify`

It intentionally does not include `next-task`.

## Quality model

The suite treats tests as more than RED/GREEN examples. Each milestone records:

- the behavior contract;
- visible red tests;
- hidden/generalization strategy;
- adequacy checks such as property/stateful tests, mutation, fuzzing,
  benchmarks, and security or schema checks;
- a risk level that determines how much validation runs before commit or merge;
- mock audit notes and real-path coverage expectations.

This keeps the resumable artifact trail while reducing the chance that an
implementation agent optimizes for visible tests instead of intended behavior.
The shared reference lives at
`plugins/agent-playbook-suite/skills/_shared/references/agentic-quality-model.md`.

## Install The Runtime CLI

The plugin provides the `docs` skill instructions, but the workflow also calls
the `docs` command. Install the published `docs-cli` package from PyPI before
using the suite:

```bash
python3 -m pip install --upgrade docs-cli
docs --version
```

Always run the newest `docs-cli`. The suite ships its bundled `docs` skill
vendored from a specific release, and that skill is generated from the same
`docs-cli` it is paired with, so the skill instructions always match the `docs`
command they call. There is no separate older floor to track: keeping the CLI
current (re-run the `--upgrade` command above) is the only supported
configuration.

Use Python 3.11 or newer. A virtual environment or `pipx` is fine as long as
the resulting `docs` command is on `PATH` for the agent session.

## Install In Codex

Codex uses a plugin marketplace catalog at `.agents/plugins/marketplace.json`.
Add this public repository as a marketplace, then install the suite plugin:

```bash
codex plugin marketplace add ArtRichards/agent-playbook-suite --ref main
codex plugin add agent-playbook-suite@agent-playbook-suite
```

To refresh the marketplace later:

```bash
python3 -m pip install --upgrade docs-cli
codex plugin marketplace upgrade agent-playbook-suite
```

If Codex already has the plugin installed and does not automatically refresh
the installed copy after a marketplace upgrade, remove and add the plugin again:

```bash
codex plugin remove agent-playbook-suite@agent-playbook-suite
codex plugin add agent-playbook-suite@agent-playbook-suite
```

## Install In Claude Code

Claude Code uses a plugin marketplace catalog at
`.claude-plugin/marketplace.json`. Add this public repository as a marketplace,
then install the suite plugin:

```bash
claude plugin marketplace add ArtRichards/agent-playbook-suite
claude plugin install agent-playbook-suite@agent-playbook-suite
```

To refresh later:

```bash
python3 -m pip install --upgrade docs-cli
claude plugin marketplace update agent-playbook-suite
claude plugin update agent-playbook-suite@agent-playbook-suite
```

Restart the agent after installing or updating plugins so the skill index is
reloaded.

## Other Agents

Gemini CLI and OpenCode do not consume the Codex or Claude plugin marketplace
manifests directly. The portable skill payload lives at
`plugins/agent-playbook-suite/skills/` for agents that support direct skill
directories. For a manual install, check out this repository and point the agent
at the relevant skill subdirectory. The six workflow skills are maintained here;
the former standalone `ArtRichards/<skill>` repositories are archived and read
only.

Use the marketplace flow above for Codex and Claude Code. Use direct skill
directories only when an agent does not support these marketplace formats.

## Repository Layout

```text
.agents/plugins/marketplace.json
.claude-plugin/marketplace.json
plugins/agent-playbook-suite/
  .codex-plugin/plugin.json
  .claude-plugin/plugin.json
  skills/
    _shared/
    docs/
    project-foundation/
    use-cases/
    create-milestones/
    ship-milestone/
    sync-and-commit/
    simplify/
```

The Codex and Claude marketplace files both point at the same suite plugin
directory. The suite plugin then exposes the shared `skills/` payload.

## Maintainer Docs

- [`marketplace-publishing-guide.md`](marketplace-publishing-guide.md) covers
  how to refresh, validate, and publish marketplace releases.
- [`GITHUB-PAGES-PUBLISHING-GUIDE.md`](GITHUB-PAGES-PUBLISHING-GUIDE.md)
  covers the GitHub Pages site.
- [`overview.md`](overview.md) explains the workflow model and how the skills
  fit together.
