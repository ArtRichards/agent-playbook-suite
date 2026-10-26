# Metadata Edits Without a docs-cli Verb

Use this policy whenever a workflow step changes a managed document's metadata
in a way no docs-cli verb performs. It is a stopgap: it applies until a
docs-cli release ships verbs for these edits and the suite pins that release.

## Which edits

docs-cli 2.0 owns every metadata change through a verb except two:

1. **A non-archived `Lifecycle:` transition** — for example promoting a
   `docs new` scaffold from `draft` to `active`, or marking a doc `done`.
   `docs archive` remains the only route into `archived`.
2. **Adding or removing a free-form `Related:` bullet on an existing
   document** — `pairs-with`, `child-of` / `parent-of`, `implements`,
   `references`, and every other verb outside the six recognized reciprocal
   ones. The six recognized verbs always go through `docs relate add/remove`.

The bundled `docs` skill forbids hand-editing a metadata block. The suite no
longer carves out an exception for these two edits: they are hand-edits, so
they need the operator's explicit approval.

## How to ask

- **Ask once per workflow run, before the first such edit.** List every
  document and the exact edit you expect the run to make (for example:
  `fetch.md`: `Lifecycle: draft` → `active`; `fetch.md`: add
  `pairs-with: fetch-impl.md`). Apply the shared operator-interaction policy.
- **An approval covers exactly the listed edits.** An edit that was not
  listed needs its own approval; never widen an approval to adjacent edits on
  the grounds that they belong to the same workflow.
- **Autonomous runs ask up front.** A workflow that otherwise runs without the
  operator (for example `ship-milestone`) collects this approval during
  milestone setup, alongside its other setup questions. If the operator is
  unreachable and no approval covers an edit, stop at that edit and return a
  decision packet to the coordinator; do not proceed by hand-editing.

## How to apply an approved edit

Change only the approved line, then run `docs touch` on the document, followed
by the workflow's usual `docs index .` and `docs check .`. Never touch any
other metadata field by hand.
