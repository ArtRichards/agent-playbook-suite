# CLAUDE.md — playbook-suite-overview

Lifecycle: active
Role: notes
Project: agent-playbook-suite
Updated: 2026-05-25

This project produces an **overview document** and a **blog post** about a suite
of five Agent Skills, originally designed for Claude Code, plus their underlying
CLI. It has no code.

## Your input

[`briefing.md`](briefing.md) is a self-contained brief covering:
- the five skills (project-foundation, create-milestones, ship-milestone,
  sync-and-commit, simplify) and their roles in the workflow;
- the underlying docs-cli convention;
- the 10-phase TDD methodology;
- the conductor + fresh-sub-agent pattern that distinguishes `ship-milestone`;
- suggested blog-post angles and pointers to read-worthy source files.

Treat the briefing as ground truth for facts (repo URLs, license, install
pattern, vocabulary, phase ordering) but do not parrot it. The overview and
blog post should each have their own voice and shape.

## Your deliverables

1. **`overview.md`** — a neutral, factual overview of the suite. Audience:
   someone deciding whether to adopt it. Should answer: what is it, what
   problem does it solve, how do the pieces fit, what is the cost of adoption,
   what is the install path. Keep it tight (target ~800-1200 words). Use the
   briefing's repo table as a starting point but feel free to restructure.

2. **`blog-post.md`** — a single blog post (pick one of the four angles in the
   briefing, or propose a fifth). Audience: developers already using
   Claude Code who want to get more leverage out of it. Voice: technical, opinionated,
   concrete, no marketing fluff. Target ~1500-2500 words. Lead with a concrete
   problem the reader recognizes; end with a clear call to action (clone these
   five repos, install docs-cli).

## Source material to consult

The briefing names the most read-worthy source files in each repo. Fetch them
directly from GitHub when you need depth on a specific point — do not invent
behavior or claim features not documented there.

Primary sources (all public, on GitHub under `ArtRichards/`):
- `docs-cli` — the underlying tool
- `project-foundation`, `create-milestones`, `ship-milestone`,
  `sync-and-commit`, `simplify` — the five skills

When in doubt about a fact, fetch the actual SKILL.md or referenced playbook
from the repo rather than guessing. Use `gh api repos/ArtRichards/<repo>/contents/<path>`
or fetch the raw URL.

## Style notes

- No emojis. No decorative characters in headers or lists.
- Code/CLI examples should be runnable as-shown.
- Cross-link repo URLs the first time each is mentioned.
- If you write something the briefing does not support, mark it clearly as
  the author's opinion or extrapolation.

## When done

Place `overview.md` and `blog-post.md` in this directory next to `briefing.md`.
No need to commit — the operator will review and publish from here.
