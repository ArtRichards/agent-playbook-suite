---
name: use-cases
description: Explore a project's primary use cases collaboratively — how a user would really use the software or tool, with best UX in mind — and record them in a use-cases doc mapped to milestone test matrices so tests focus on use cases first. Runs automatically after `project-foundation` completes (optional, but strongly preferred); also invoke directly when discussing use cases or when feedback work surfaces use-case information. Triggers on "explore use cases", "what are the use cases", "how will users use this", "discuss use cases".
---

# use-cases

Work with the operator to understand what the project's primary
use cases are and how a user should use the product, highlight
those as primary goals, and record them in a `use-cases.md` doc
whose entries map to milestone test matrices. Tests focus on
these use cases primarily.

Requires [`docs-cli`](https://github.com/ArtRichards/docs-cli) —
the doc is authored via `docs new` and maintained with docs
verbs, like every other artifact in the tree.

## When this applies

- **Automatically after `project-foundation` completes** (the
  Definition of Ready is `active`). Optional, but strongly
  preferred — milestone work functions without a use-cases doc,
  but tests lose their primary semantic anchor.
- **Directly**, whenever the operator wants to discuss use cases
  or how users will work with the product.
- **During feedback work**, when feedback reveals new use-case
  information — update the existing doc rather than starting
  over.

Do **not** apply when the project has no docs tree yet — run
`project-foundation` first.

## Substance lives in references/

- [`references/use-cases-playbook.md`](references/use-cases-playbook.md)
  — the collaborative exploration procedure and the use-cases
  doc template.

## Invariants

1. **Explore, don't interrogate.** Propose candidate use cases
   from the foundation docs and your own product sense — never
   just demand use cases from the operator. Work together toward
   how a user could really utilize the software, with best UX in
   mind. The operator corrects concrete proposals; they do not
   fill in a blank form.
2. **Primary use cases are primary goals.** Milestone test
   matrices map against them, and tests focus on demonstrating
   them first — semantic behavior over incidental representation.
3. **The doc is docs-managed.** Author with `docs new`, bump with
   `docs touch`, validate with `docs check`. Never hand-edit
   metadata.
4. **Optional, but strongly preferred.** If milestone work starts
   without the doc, nudge once toward running this skill — do not
   block.
