# Marketplace Publishing Guide

Lifecycle: active
Role: guide
Project: agent-playbook-suite
Updated: 2026-06-01

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

The payload may also include `skills/_shared/` for shared references such as
the agentic quality model. `_shared` is not a skill and must not contain a
`SKILL.md`.

Do not add `next-task` to the suite.

## Refresh The Skill Payload

Policy: the suite always tracks the newest `docs-cli` release and keeps its
bundled `docs` skill in lockstep with that release. Each published suite release
records the exact `docs-cli` version it was vendored from and tested against —
the pin — so a version bump is a visible, reviewable diff rather than an implicit
"whatever PyPI served at CI time".

Install or upgrade the runtime CLI from PyPI first:

```bash
python3 -m pip install --upgrade docs-cli
docs --version
```

Refresh the bundled `docs` skill from the installed PyPI package so it matches
the version you just installed:

```bash
docs install-skill --dest plugins/agent-playbook-suite/skills/docs --force
```

Then move the tested-version pin forward to the version you just vendored. The
single source of truth is the `DOCS_CLI_VERSION` value in
[`.github/workflows/validate.yml`](.github/workflows/validate.yml); update it to
match `docs --version`. CI installs that exact version and fails if the installed
binary drifts from the pin, so the pin and the vendored `docs` skill can never
silently fall out of sync. Always bump the pin to the newest release rather than
holding it back.

The five workflow skills — `project-foundation`, `create-milestones`,
`ship-milestone`, `sync-and-commit`, and `simplify` — are maintained directly in
this repository under `plugins/agent-playbook-suite/skills/`. This repository is
their source of truth: edit them in place. The standalone
`ArtRichards/<skill>` repositories are archived and read-only; they exist only as
historical pointers back here. Do not rsync or copy from them — their content
predates the risk-aware upgrade, so pulling it in would silently revert the
skills.

Only the `docs` skill is vendored from an external source: the `docs-cli` PyPI
package, via `docs install-skill` above.

After refresh, confirm the payload contains only the intended skill set plus
optional shared references:

```bash
find plugins/agent-playbook-suite/skills -mindepth 1 -maxdepth 1 -type d -printf '%f\n' | sort
find plugins/agent-playbook-suite/skills -name .git -print
```

The first command should print the six skill directories and, if present,
`_shared`. The second command should print nothing. Each skill directory must
contain `SKILL.md`; `_shared` must only contain reusable references.

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

The `Validate suite` workflow in `.github/workflows/validate.yml` is the
release gate. Run the same checks locally when possible before publishing.

Validate JSON syntax:

```bash
python3 -m json.tool .agents/plugins/marketplace.json >/dev/null
python3 -m json.tool .claude-plugin/marketplace.json >/dev/null
python3 -m json.tool plugins/agent-playbook-suite/.codex-plugin/plugin.json >/dev/null
python3 -m json.tool plugins/agent-playbook-suite/.claude-plugin/plugin.json >/dev/null
```

Validate the skill payload shape:

```bash
python3 - <<'PY'
from pathlib import Path

expected = {
    "create-milestones",
    "docs",
    "project-foundation",
    "ship-milestone",
    "simplify",
    "sync-and-commit",
}
skills = Path("plugins/agent-playbook-suite/skills")
actual = {p.name for p in skills.iterdir() if p.is_dir()}
unexpected = actual - expected - {"_shared"}
missing = expected - actual
if missing or unexpected:
    raise SystemExit(f"Skill payload mismatch: missing={sorted(missing)} unexpected={sorted(unexpected)}")
for skill in expected:
    skill_file = skills / skill / "SKILL.md"
    if not skill_file.exists():
        raise SystemExit(f"Missing {skill_file}")
if (skills / "_shared" / "SKILL.md").exists():
    raise SystemExit("_shared must not be exposed as a skill")
nested_git = list(skills.rglob(".git"))
if nested_git:
    raise SystemExit(f"Nested .git dirs are not allowed: {nested_git}")
PY
```

Validate quality-model coverage:

```bash
python3 - <<'PY'
from pathlib import Path

skills = Path("plugins/agent-playbook-suite/skills")
required = {
    "project-foundation": ("hidden/generalization", "risk level", "mock"),
    "create-milestones": ("hidden/generalization", "mock", "mutation"),
    "ship-milestone": ("hidden/generalization", "mock", "mutation"),
    "sync-and-commit": ("hidden/generalization", "mock", "mutation", "risk level"),
    "simplify": ("hidden/generalization", "mock", "mutation", "risk level"),
    "docs": ("quality artifacts", "docs check", "generated reports", "mutation"),
}
shared = skills / "_shared" / "references" / "agentic-quality-model.md"
if not shared.exists():
    raise SystemExit(f"Missing {shared}")
for skill, terms in required.items():
    text = "\n".join(p.read_text(encoding="utf-8") for p in (skills / skill).rglob("*.md")).lower()
    missing = [term for term in terms if term not in text]
    if missing:
        raise SystemExit(f"{skill} missing quality-model terms: {missing}")
PY
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

Build the GitHub Pages site when Ruby dependencies are available:

```bash
cd site
bundle exec jekyll build
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

For both smoke tests, also confirm:

- `docs`, `project-foundation`, `create-milestones`, `ship-milestone`,
  `sync-and-commit`, and `simplify` are installed;
- `_shared` is present only as a reference directory if installed;
- the shared quality model is readable from installed workflow skills;
- the bundled `docs` skill exposes quality-artifact guidance.

## Publish

Before publishing, require:

- `Validate suite` workflow green;
- GitHub Pages/site build green where applicable;
- `docs index` and `docs check` green;
- marketplace JSON and both plugin manifests parse;
- skill payload contains exactly the intended skill directories plus optional
  `_shared`;
- quality-model coverage checks pass;
- install smoke test complete for Codex and Claude Code when both are
  supported.

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
