# Marketplace Publishing Guide

Lifecycle: active
Role: guide
Project: agent-playbook-suite
Updated: 2026-08-16

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
- `use-cases`
- `explore`
- `manage-milestone-tracker`
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
binary drifts from the pin. The maintainer refresh step above, not a byte-for-byte
CI comparison, keeps the vendored `docs` skill aligned with that release. Always
bump the pin to the newest release rather than holding it back.

The eight workflow skills — `project-foundation`, `use-cases`, `explore`,
`manage-milestone-tracker`, `create-milestones`, `ship-milestone`,
`sync-and-commit`, and `simplify` — are
maintained directly in this repository under
`plugins/agent-playbook-suite/skills/`. This repository is
their source of truth: edit them in place. The five formerly standalone
workflow-skill repositories are archived and read-only; they exist only as
historical pointers back here. `use-cases`, `explore`, and
`manage-milestone-tracker` originate in this suite. Do not rsync or copy from
the archived repositories — their content
predates the risk-aware upgrade, so pulling it in would silently revert skills.

Only the `docs` skill is vendored from an external source: the `docs-cli` PyPI
package, via `docs install-skill` above.

After refresh, confirm the payload contains only the intended skill set plus
optional shared references:

```bash
find plugins/agent-playbook-suite/skills -mindepth 1 -maxdepth 1 -type d -printf '%f\n' | sort
find plugins/agent-playbook-suite/skills -name .git -print
```

The first command should print the nine skill directories and, if present,
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
    "explore",
    "manage-milestone-tracker",
    "project-foundation",
    "ship-milestone",
    "simplify",
    "sync-and-commit",
    "use-cases",
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

Validate generated skill interface metadata with the same pinned parser as CI:

```bash
python3 -m pip install "PyYAML==6.0.2"
python3 - <<'PY'
from pathlib import Path
import yaml

for skill in ("explore", "manage-milestone-tracker"):
    path = Path("plugins/agent-playbook-suite/skills") / skill / "agents" / "openai.yaml"
    data = yaml.safe_load(path.read_text(encoding="utf-8"))
    interface = data.get("interface") if isinstance(data, dict) else None
    if not isinstance(interface, dict):
        raise SystemExit(f"{path} must contain an interface mapping")
    for key in ("display_name", "short_description", "default_prompt"):
        value = interface.get(key)
        if not isinstance(value, str) or not value.strip():
            raise SystemExit(f"{path} missing non-empty interface.{key}")
    if not 25 <= len(interface["short_description"]) <= 64:
        raise SystemExit(f"{path} short_description must be 25-64 characters")
    if f"${skill}" not in interface["default_prompt"]:
        raise SystemExit(f"{path} default_prompt must mention ${skill}")

prompt = yaml.safe_load(
    Path("plugins/agent-playbook-suite/skills/explore/agents/openai.yaml")
    .read_text(encoding="utf-8")
)["interface"]["default_prompt"].lower()
required_terms = (
    "compare",
    "feasibility",
    "recovery",
    "evidence-backed",
    "exact unresolved gap",
)
missing = [term for term in required_terms if term not in prompt]
if missing:
    raise SystemExit(f"explore default_prompt missing modes/outcomes: {missing}")
PY
```

Validate quality-model coverage:

```bash
python3 - <<'PY'
from pathlib import Path

skills = Path("plugins/agent-playbook-suite/skills")
required = {
    "explore": (
        "solution uncertainty",
        "pattern-preservation",
        "simplicity gate",
        "operator-owned product decision",
        "no viable route",
        "contract owner",
    ),
    "project-foundation": ("hidden/generalization", "risk level", "mock"),
    "create-milestones": ("hidden/generalization", "mock", "mutation"),
    "ship-milestone": ("hidden/generalization", "mock", "mutation"),
    "sync-and-commit": ("hidden/generalization", "mock", "mutation", "risk level"),
    "simplify": ("hidden/generalization", "mock", "mutation", "risk level"),
    "docs": ("quality artifacts", "docs check", "generated reports", "mutation"),
    "use-cases": ("use case", "test matrices"),
}
shared = skills / "_shared" / "references" / "agentic-quality-model.md"
if not shared.exists():
    raise SystemExit(f"Missing {shared}")
shared_text = shared.read_text(encoding="utf-8").lower()
for term in (
    "contract layer",
    "visible red tests",
    "hidden/generalization",
    "adequacy tests",
    "risk levels",
    "mock policy",
    "forbidden shortcuts",
    "solution uncertainty",
):
    if term not in shared_text:
        raise SystemExit(f"Shared quality model missing term: {term}")
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

The system `plugin-creator` validator treats every directory below `skills/` as
a skill, while this bundle intentionally keeps non-skill references in
`skills/_shared/`. When that validator is available, run it against a disposable
staged copy without `_shared`; the actual-tree payload checks above and the
marketplace smoke test still own the `_shared` invariant and installed-reference
check:

```bash
tmp="$(mktemp -d)"
cp -R plugins/agent-playbook-suite "$tmp/plugin"
rm -rf "$tmp/plugin/skills/_shared"
python3 "${CODEX_HOME:-$HOME/.codex}/skills/.system/plugin-creator/scripts/validate_plugin.py" "$tmp/plugin"
rm -rf "$tmp"
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

Restart the relevant agent and confirm the eight workflow skills plus `docs`
are discoverable.

For both smoke tests, also confirm:

- `docs`, `project-foundation`, `use-cases`, `explore`,
  `manage-milestone-tracker`, `create-milestones`, `ship-milestone`,
  `sync-and-commit`, and `simplify` are installed;
- `_shared` is present as a required reference directory in the full plugin
  install and is not exposed as an invokable skill;
- the shared quality model, milestone-tracker contract, and
  operator-interaction policy are readable from installed workflow skills;
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
updates together on a release branch. Do not push a release commit directly to
`main`; use a CI-backed pull request so the repository follows the same branch
safety rule that the published `sync-and-commit` skill teaches:

```bash
VERSION=$(python3 -c 'import json; print(json.load(open("plugins/agent-playbook-suite/.codex-plugin/plugin.json"))["version"])')
git switch -c "release/$VERSION"
git status --short
git add \
  .agents/plugins/marketplace.json \
  .claude-plugin/marketplace.json \
  .github/workflows \
  AGENTS.md CLAUDE.md INDEX.md README.md \
  blog-post.md briefing.md docs marketplace-publishing-guide.md overview.md \
  plugins/agent-playbook-suite site
git commit -m "Release Agent Playbook Suite $VERSION"
RELEASE_HEAD=$(git rev-parse HEAD)
git push -u origin "release/$VERSION"
gh pr create \
  --base main \
  --head "release/$VERSION" \
  --title "Release Agent Playbook Suite $VERSION" \
  --body "Publish the validated Agent Playbook Suite $VERSION payload and docs."
gh pr checks --watch
gh pr merge --merge --delete-branch --match-head-commit "$RELEASE_HEAD"
git switch main
git pull --ff-only
```

After the merged release commit and the `main` validation plus Pages workflows
are green, tag that exact local `main` commit to match the manifest version and
push only the tag:

```bash
VERSION=$(python3 -c 'import json; print(json.load(open("plugins/agent-playbook-suite/.codex-plugin/plugin.json"))["version"])')
git tag -a "v$VERSION" -m "Agent Playbook Suite v$VERSION"
git push origin "v$VERSION"
```

Use an annotated tag (`-a`); the tag name (`vX.Y.Z`) must equal the version in
both plugin manifests and `.claude-plugin/marketplace.json`. The marketplace
installs from `main`, so the tag is a release marker for history and rollback,
not the install source.

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
