# Agent Skill Installation

Lifecycle: active
Role: notes
Project: agent-playbook-suite
Updated: 2026-05-25

This file is the source of truth for installing the Agent Playbook Suite from
its public sources. It covers **Claude Code**, **Codex**, **Gemini CLI**, and
**OpenCode**. Keep the skill list here current first, then let overview or
blog-post copy refer back to this file instead of duplicating install commands.

Each agent discovers user skills from a per-agent directory:

- **Claude Code** — `~/.claude/skills/`
- **Codex** — `$CODEX_HOME/skills`, defaulting to `~/.codex/skills`
- **Gemini CLI** — `~/.gemini/skills/` or the shared `~/.agents/skills/` alias
- **OpenCode** — `~/.config/opencode/skills/`, `~/.claude/skills/`, or the shared `~/.agents/skills/` alias

Each installed skill is a directory in that folder with a `SKILL.md` at its
root. The install shapes below produce that layout for all supported agents.

## Canonical Skill Set

| Skill | Public source | Install shape |
|---|---|---|
| `docs` | PyPI package `docs-cli` | Install the published `docs-cli` package with `python -m pip install --upgrade docs-cli`, then materialize the bundled skill with `docs install-skill --dest <skills-dir>/docs`. |
| `project-foundation` | `https://github.com/ArtRichards/project-foundation` | Clone the repo into the agent's skills directory. |
| `create-milestones` | `https://github.com/ArtRichards/create-milestones` | Clone the repo into the agent's skills directory. |
| `ship-milestone` | `https://github.com/ArtRichards/ship-milestone` | Clone the repo into the agent's skills directory. |
| `sync-and-commit` | `https://github.com/ArtRichards/sync-and-commit` | Clone the repo into the agent's skills directory. |
| `simplify` | `https://github.com/ArtRichards/simplify` | Clone the repo into the agent's skills directory. |

The five workflow skills are root-level skill repositories. Install them as git
checkouts so later updates are ordinary `git pull --ff-only` operations.

The `docs` skill is different: it is bundled in the published `docs-cli` Python
package rather than installed as a separate skill repository. The supported
install path is to install `docs-cli` from PyPI, then run `docs install-skill`
for each agent-specific skill directory. The other suite skills call the
`docs` command, so the PyPI package must be installed anywhere the suite runs.
Use your normal Python packaging workflow (`python -m pip`, `pipx`, or a
virtual environment), but make sure the resulting `docs` command is on `PATH`.

Restart the agent after installing new skills so it reloads the skill index.

Verify the CLI before installing agent skills:

```bash
docs --version
```

---

## Claude Code

Claude Code auto-discovers skills from `~/.claude/skills/`. There is no
dedicated installer; a skill is "installed" when its directory exists at that
path with a `SKILL.md` at its root.

### Fresh Install

Use this on a machine where these skill directories do not already exist:

```bash
export CLAUDE_SKILLS_DIR="$HOME/.claude/skills"
mkdir -p "$CLAUDE_SKILLS_DIR"

python -m pip install --upgrade docs-cli
docs install-skill --dest "$CLAUDE_SKILLS_DIR/docs"

for skill in \
  project-foundation \
  create-milestones \
  ship-milestone \
  sync-and-commit \
  simplify
do
  git clone "https://github.com/ArtRichards/$skill" "$CLAUDE_SKILLS_DIR/$skill"
done
```

### Safe Update

Use this when the workflow skill directories are already git checkouts:

```bash
export CLAUDE_SKILLS_DIR="$HOME/.claude/skills"
mkdir -p "$CLAUDE_SKILLS_DIR"

python -m pip install --upgrade docs-cli
docs install-skill --dest "$CLAUDE_SKILLS_DIR/docs" --force

for skill in \
  project-foundation \
  create-milestones \
  ship-milestone \
  sync-and-commit \
  simplify
do
  dest="$CLAUDE_SKILLS_DIR/$skill"
  if [ -d "$dest/.git" ]; then
    git -C "$dest" pull --ff-only
  elif [ -e "$dest" ]; then
    echo "Skipping $dest because it exists but is not a git checkout."
  else
    git clone "https://github.com/ArtRichards/$skill" "$dest"
  fi
done
```

If a workflow skill directory exists but is not a git checkout, inspect or back
it up before replacing it. It may contain local edits or an older manually
installed copy.

### Replace Older Copied Installs

Use this when existing workflow skill directories are copied folders rather
than git checkouts and you want to replace them with public repo installs:

```bash
export CLAUDE_SKILLS_DIR="$HOME/.claude/skills"
backup_root="$HOME/.claude/skills-backup-$(date +%Y%m%d-%H%M%S)"
mkdir -p "$CLAUDE_SKILLS_DIR" "$backup_root"

for skill in \
  project-foundation \
  create-milestones \
  ship-milestone \
  sync-and-commit \
  simplify
do
  dest="$CLAUDE_SKILLS_DIR/$skill"
  if [ -e "$dest" ]; then
    mv "$dest" "$backup_root/$skill"
  fi
  git clone "https://github.com/ArtRichards/$skill" "$dest"
done

echo "Backed up replaced skills to $backup_root"
```

This intentionally backs up before replacing. Delete the backup only after
confirming the new skills load correctly.

---

## Codex

Codex reads user skills from `$CODEX_HOME/skills`. If `CODEX_HOME` is unset,
that resolves to `~/.codex/skills`.

### Fresh Install

Use this on a machine where these skill directories do not already exist:

```bash
export CODEX_HOME="${CODEX_HOME:-$HOME/.codex}"
export CODEX_SKILLS_DIR="$CODEX_HOME/skills"
mkdir -p "$CODEX_SKILLS_DIR"

python -m pip install --upgrade docs-cli
docs install-skill --dest "$CODEX_SKILLS_DIR/docs"

for skill in \
  project-foundation \
  create-milestones \
  ship-milestone \
  sync-and-commit \
  simplify
do
  git clone "https://github.com/ArtRichards/$skill" "$CODEX_SKILLS_DIR/$skill"
done
```

### Safe Update

Use this when the workflow skill directories are already git checkouts:

```bash
export CODEX_HOME="${CODEX_HOME:-$HOME/.codex}"
export CODEX_SKILLS_DIR="$CODEX_HOME/skills"
mkdir -p "$CODEX_SKILLS_DIR"

python -m pip install --upgrade docs-cli
docs install-skill --dest "$CODEX_SKILLS_DIR/docs" --force

for skill in \
  project-foundation \
  create-milestones \
  ship-milestone \
  sync-and-commit \
  simplify
do
  dest="$CODEX_SKILLS_DIR/$skill"
  if [ -d "$dest/.git" ]; then
    git -C "$dest" pull --ff-only
  elif [ -e "$dest" ]; then
    echo "Skipping $dest because it exists but is not a git checkout."
  else
    git clone "https://github.com/ArtRichards/$skill" "$dest"
  fi
done
```

If a workflow skill directory exists but is not a git checkout, inspect or back
it up before replacing it. It may contain local edits or an older manually
installed copy.

### Replace Older Copied Installs

Use this when existing workflow skill directories are copied folders rather
than git checkouts and you want to replace them with public repo installs:

```bash
export CODEX_HOME="${CODEX_HOME:-$HOME/.codex}"
export CODEX_SKILLS_DIR="$CODEX_HOME/skills"
backup_root="$CODEX_HOME/skills-backup-$(date +%Y%m%d-%H%M%S)"
mkdir -p "$CODEX_SKILLS_DIR" "$backup_root"

for skill in \
  project-foundation \
  create-milestones \
  ship-milestone \
  sync-and-commit \
  simplify
do
  dest="$CODEX_SKILLS_DIR/$skill"
  if [ -e "$dest" ]; then
    mv "$dest" "$backup_root/$skill"
  fi
  git clone "https://github.com/ArtRichards/$skill" "$dest"
done

echo "Backed up replaced skills to $backup_root"
```

This intentionally backs up before replacing. Delete the backup only after
confirming the new skills load correctly.

### Codex GitHub Installer Alternative

Codex ships a helper that can download a public GitHub repo path into
`$CODEX_HOME/skills`. It is useful for one-off workflow-skill installs, but it
copies files instead of creating git checkouts. For workflow skills, prefer
`git clone` if you want easy future updates. Do not use this helper for the
`docs` skill; install `docs-cli` from PyPI and run `docs install-skill`.

Install a root-level workflow skill with the helper:

```bash
export CODEX_HOME="${CODEX_HOME:-$HOME/.codex}"

python "$CODEX_HOME/skills/.system/skill-installer/scripts/install-skill-from-github.py" \
  --repo ArtRichards/ship-milestone \
  --path . \
  --name ship-milestone
```

The helper aborts if the destination directory already exists. Use the safe
update or replacement flows above for ongoing maintenance.

---

## Gemini CLI

Gemini CLI discovers user skills from `~/.gemini/skills/` and from the shared
`~/.agents/skills/` alias. Workspace skills can live in `.gemini/skills/` or
`.agents/skills/`. Use the native Gemini path below unless you intentionally
want a shared user-level skill library.

### Fresh Install

Use this on a machine where these skill directories do not already exist:

```bash
export GEMINI_SKILLS_DIR="$HOME/.gemini/skills"
mkdir -p "$GEMINI_SKILLS_DIR"

python -m pip install --upgrade docs-cli
docs install-skill --dest "$GEMINI_SKILLS_DIR/docs"

for skill in \
  project-foundation \
  create-milestones \
  ship-milestone \
  sync-and-commit \
  simplify
do
  git clone "https://github.com/ArtRichards/$skill" "$GEMINI_SKILLS_DIR/$skill"
done
```

Gemini CLI also provides `gemini skills install` for installing skills from
Git repositories or local directories. Direct git checkouts are still the
simplest updateable shape for this suite.

### Safe Update

Use the same safe update pattern as Claude Code, substituting
`GEMINI_SKILLS_DIR="$HOME/.gemini/skills"`.

### Shared User Path

If you want Gemini CLI and OpenCode to read the same user-level copies, install
the suite into `~/.agents/skills/` instead:

```bash
export AGENT_SKILLS_DIR="$HOME/.agents/skills"
mkdir -p "$AGENT_SKILLS_DIR"

python -m pip install --upgrade docs-cli
docs install-skill --dest "$AGENT_SKILLS_DIR/docs"

for skill in \
  project-foundation \
  create-milestones \
  ship-milestone \
  sync-and-commit \
  simplify
do
  git clone "https://github.com/ArtRichards/$skill" "$AGENT_SKILLS_DIR/$skill"
done
```

---

## OpenCode

OpenCode discovers skills from native OpenCode paths and from compatible Claude
Code and `.agents` paths. Native global skills live in
`~/.config/opencode/skills/`; project-local skills can live in
`.opencode/skills/`. OpenCode also reads `~/.claude/skills/`,
`.claude/skills/`, `~/.agents/skills/`, and `.agents/skills/`.

### Fresh Install

Use this for native global OpenCode installation:

```bash
export OPENCODE_SKILLS_DIR="$HOME/.config/opencode/skills"
mkdir -p "$OPENCODE_SKILLS_DIR"

python -m pip install --upgrade docs-cli
docs install-skill --dest "$OPENCODE_SKILLS_DIR/docs"

for skill in \
  project-foundation \
  create-milestones \
  ship-milestone \
  sync-and-commit \
  simplify
do
  git clone "https://github.com/ArtRichards/$skill" "$OPENCODE_SKILLS_DIR/$skill"
done
```

### Safe Update

Use the same safe update pattern as Claude Code, substituting
`OPENCODE_SKILLS_DIR="$HOME/.config/opencode/skills"`.

### Shared User Path

If you already installed the suite into `~/.agents/skills/` for Gemini CLI or
another agent, OpenCode can read that same user-level directory. That is the
best option when you want one shared skill checkout across tools that support
the `.agents` alias.
