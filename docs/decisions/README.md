# Architecture Decision Records

This is the append-only ledger of **why** things changed. `PRD.md` and
`ARCHITECTURE.md` describe the current state; each record here is the immutable,
dated account of a decision that produced it.

The perishable part of a decision is not the outcome, it is the drivers and the
options that lost. Those live nowhere but here.

## When to write one

Only when a change **reverses or materially alters** a stated invariant or a
commitment in `PRD.md` or `ARCHITECTURE.md`, the kind of thing where future-you
asks "wait, why did we do it this way?". Pure additions and clarifications are
just doc edits, not ADRs. (See the Workflow section of [`AGENTS.md`](../../AGENTS.md).)

## How

- Copy [`template.md`](template.md) to `NNNN-slug.md`, zero-padded, next number in sequence.
- Format is [MADR](https://adr.github.io/madr/). Fill the sections; delete none.
- **Also edit the canonical doc** (`PRD.md` / `ARCHITECTURE.md` / `GLOSSARY.md`)
  to reflect the new current state, and link it back here (`see ADR-NNNN`).
- Records are **append-only**. Never edit an `Accepted` record to reverse it.
  Write a new record that supersedes it, and set the old one's status to
  `Superseded by ADR-NNNN`.

| Status | Meaning |
|---|---|
| `Proposed` | Written to hold a comparison; no outcome chosen yet. Not in force. |
| `Accepted` | In force. |
| `Rejected` | Proposed and turned down, so never in force. Retained because the drivers and the evidence are the perishable part, and a rejected option gets reproposed otherwise. |
| `Superseded by ADR-NNNN` | Replaced; retained for the audit trail, never deleted. |

## Index

| ADR | Decision | Status |
|---|---|---|
| [0001](0001-distribute-as-claude-code-plugins.md) | Distribute as Claude Code plugins through a marketplace | Accepted |
| [0002](0002-one-plugin-per-unit-grouped-by-cohesion.md) | One plugin per unit, grouped by cohesion | Accepted |
| [0003](0003-share-between-plugins-with-manifest-dependencies.md) | Share between plugins with manifest dependencies | Accepted |
| [0004](0004-own-the-skills-we-use.md) | Own the skills we use rather than depend on a third-party collection | Accepted |
| [0005](0005-author-from-a-blank-page.md) | Author skills from a blank page, treating prior art as reference | Accepted |
| [0006](0006-declare-invoked-skills-as-dependencies.md) | Declare the skills an agent invokes as plugin dependencies | Accepted |
| [0007](0007-private-until-switchover.md) | Keep the repository private until switchover | Accepted |
| [0008](0008-document-with-current-state-docs-and-a-ledger.md) | Document the repo with current-state docs and an append-only ledger | Accepted |
| [0009](0009-own-the-skill-writing-standard.md) | Own the skill-writing standard as a thin skill | Accepted |
| [0010](0010-extract-the-review-baseline-into-a-library-skill.md) | Extract the review baseline into a shared library skill | Accepted |
| [0011](0011-docs-must-reconstruct-the-repo.md) | The docs must be able to reconstruct the repo | Superseded by ADR-0012 |
| [0012](0012-specify-per-component-not-per-repo.md) | Specify per component, not per repo | Accepted |
| [0013](0013-keep-plugins-in-a-plugins-directory.md) | Keep plugins in a `plugins/` directory | Accepted |
| [0014](0014-plugin-specs-live-in-the-plugin.md) | A plugin's spec lives in the plugin | Superseded by ADR-0015 |
| [0015](0015-specs-are-one-file-per-plugin-under-docs.md) | Specs are one file per plugin, under `docs/specs/` | Accepted |
| [0016](0016-call-skills-by-their-namespaced-name.md) | Call skills by their namespaced name | Accepted |
| [0017](0017-restructure-around-a-flat-skills-tree.md) | Restructure around a flat `skills/` tree | Rejected |
