# Marketplace Publishing Guide

Lifecycle: active
Role: guide
Project: agent-playbook-suite
Updated: 2026-05-25

This guide is the maintainer checklist for publishing Agent Playbook Suite as
one plugin marketplace package for Codex and Claude Code.

The public install surface is:

- Codex marketplace: `.agents/plugins/marketplace.json`
- Claude Code marketplace: `.claude-plugin/marketplace.json`
- Shared suite plugin: `plugins/agent-playbook-suite/`
- Shared skill payload: `plugins/agent-playbook-suite/skills/`

The suite contains exactly these skills:

- `docs`
- `project-foundation`
- `create-milestones`
- `ship-milestone`
- `sync-and-commit`
- `simplify`

Do not add `next-task` to the suite.

## Refresh The Skill Payload

Install or upgrade the runtime CLI from PyPI first:

```bash
python3 -m pip install --upgrade docs-cli
docs --version
```

Refresh the bundled `docs` skill from the installed PyPI package:

```bash
docs install-skill --dest plugins/agent-playbook-suite/skills/docs --force
```

Refresh each workflow skill from a clean checkout of its public source repo.
Set `WORKFLOW_SKILLS_DIR` to the parent directory that contains those checkouts.

```bash
export WORKFLOW_SKILLS_DIR="$HOME/src"

rsync -a --delete --exclude .git "$WORKFLOW_SKILLS_DIR/project-foundation/" plugins/agent-playbook-suite/skills/project-foundation/
rsync -a --delete --exclude .git "$WORKFLOW_SKILLS_DIR/create-milestones/" plugins/agent-playbook-suite/skills/create-milestones/
rsync -a --delete --exclude .git "$WORKFLOW_SKILLS_DIR/ship-milestone/" plugins/agent-playbook-suite/skills/ship-milestone/
rsync -a --delete --exclude .git "$WORKFLOW_SKILLS_DIR/sync-and-commit/" plugins/agent-playbook-suite/skills/sync-and-commit/
rsync -a --delete --exclude .git "$WORKFLOW_SKILLS_DIR/simplify/" plugins/agent-playbook-suite/skills/simplify/
```

After refresh, confirm the payload contains only the intended skill set:

```bash
find plugins/agent-playbook-suite/skills -mindepth 1 -maxdepth 1 -type d -printf '%f\n' | sort
find plugins/agent-playbook-suite/skills -name .git -print
```

The second command should print nothing.

## Update Marketplace Metadata

When publishing a real release, update the version in both plugin manifests:

- `plugins/agent-playbook-suite/.codex-plugin/plugin.json`
- `plugins/agent-playbook-suite/.claude-plugin/plugin.json`

Keep the marketplace entries aligned with that version where the marketplace
schema includes a version field:

- `.claude-plugin/marketplace.json`

The Codex marketplace entry points at `./plugins/agent-playbook-suite` and
uses the plugin manifest for version and package metadata.

## Validate Locally

Validate JSON syntax:

```bash
python3 -m json.tool .agents/plugins/marketplace.json >/dev/null
python3 -m json.tool .claude-plugin/marketplace.json >/dev/null
python3 -m json.tool plugins/agent-playbook-suite/.codex-plugin/plugin.json >/dev/null
python3 -m json.tool plugins/agent-playbook-suite/.claude-plugin/plugin.json >/dev/null
```

Validate the Claude marketplace and plugin:

```bash
claude plugin validate .
```

Refresh this docs tree:

```bash
docs index
docs check .
```

## Smoke Test Marketplace Installs

Use a disposable profile or a machine where replacing the local test install is
acceptable.

For Codex:

```bash
codex plugin marketplace add .
codex plugin add agent-playbook-suite@agent-playbook-suite
```

`codex plugin marketplace upgrade` only applies to Git-backed marketplace
sources. Do not run it for the local smoke-test marketplace.

For Claude Code:

```bash
claude plugin marketplace add ./
claude plugin install agent-playbook-suite@agent-playbook-suite
```

Restart the relevant agent and confirm the six skills are discoverable.

## Publish

Commit the marketplace files, plugin manifests, skill payload, README, and docs
updates together:

```bash
git status --short
git add .
git commit -m "Publish agent playbook suite marketplace"
git push origin main
```

The repository is the marketplace source. After the push, users can install
from GitHub with:

```bash
codex plugin marketplace add ArtRichards/agent-playbook-suite --ref main
codex plugin add agent-playbook-suite@agent-playbook-suite

claude plugin marketplace add ArtRichards/agent-playbook-suite
claude plugin install agent-playbook-suite@agent-playbook-suite
```

GitHub Pages publishing is separate. Use
[`GITHUB-PAGES-PUBLISHING-GUIDE.md`](GITHUB-PAGES-PUBLISHING-GUIDE.md) when the
website needs to be updated or verified.
