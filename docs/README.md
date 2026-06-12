# Docs Layout

Lifecycle: draft
Role: guide
Project: agent-playbook-suite
Updated: 2026-06-08

## Purpose

This directory holds internal project-management documentation for Agent Playbook Suite. Keep public entry points at the repository root, and put process specs, design notes, and future planning artifacts here when they are mainly for maintaining the suite itself.

## Current Layout

- `docs/specs/`: process and design specs for changing the suite.
- `docs/specs/<topic>/`: a topic directory for a spec and its companion
  artifacts, such as logs, examples, decision notes, or validation records.

Root-level Markdown remains the public surface unless a future reorganization deliberately moves it:

- `README.md`: install guide and top-level product summary.
- `overview.md`: public workflow overview.
- `blog-post.md`: public long-form article source.
- `marketplace-publishing-guide.md`: maintainer publishing checklist.
- `AGENTS.md` and `CLAUDE.md`: agent context for this repository.

## Placement Rules

Use `docs/specs/` for specs that define how the suite should evolve, how feedback should be handled, or how new capabilities should be evaluated. When a spec has companion artifacts, group them in a topic directory such as `docs/specs/feedback-integration/`.

Avoid role-based folders such as `docs/logs/` unless the project deliberately adopts them. `docs-cli` does not treat `logs/` specially; `Role: log` in the metadata is the thing that makes a document a log.

Keep files at the repository root when they are public entry points, host instructions, marketplace guides, or compatibility material that users expect to find immediately after opening the repo.

Do not move files into this directory by hand. Use `docs mv` so `Related:` references and `INDEX.md` stay consistent.

## Validation

After adding or moving managed docs, run:

```sh
docs index .
docs check . --stale 14
```
