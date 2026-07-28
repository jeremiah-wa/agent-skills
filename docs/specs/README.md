# agent-skills, Component Specifications

Every component in this repo carries a spec. This file says what a spec must
contain; the specs themselves are one file per plugin in this directory
([ADR-0015](../decisions/0015-specs-are-one-file-per-plugin-under-docs.md)).

This is the companion to [`ARCHITECTURE.md`](../ARCHITECTURE.md) (which gives the
shape) and [`GLOSSARY.md`](../GLOSSARY.md) (which gives the vocabulary). This file
is canonical for the **shape** of a contract; each plugin's spec is canonical for
its own components.

## The rule

**No component ships unspecified.** A plugin without a spec here, or a component
absent from its plugin's spec, is incomplete regardless of whether it works.

The bar is per component: specified to the level another agent needs to re-author
it from a blank page ([ADR-0012](../decisions/0012-specify-per-component-not-per-repo.md)).
Two bodies that satisfy one contract are both correct, because
[ADR-0005](../decisions/0005-author-from-a-blank-page.md) re-authors expression
rather than copying it. The spec is the requirement, never the wording.

## What a spec contains

`docs/specs/<plugin>.md` opens with the plugin, then one section per component.
One file per plugin, because the plugin is the unit of install and versioning
and a spec tracks its `version` bump.

**For the plugin:**

- What it is for, in a sentence, and why it is one plugin rather than several
  (the cohesion rule, [ADR-0002](../decisions/0002-one-plugin-per-unit-grouped-by-cohesion.md)).
- Its per-plugin manifest values: `name`, `version`, `description`, `keywords`,
  and `skills` and `dependencies` where they apply. The mechanical fields are
  fixed by the template in [`AGENTS.md`](../../AGENTS.md#adding-a-plugin) and are
  not restated.

**For each component** (skill, agent, or reference file):

- **Path and frontmatter**: where the file goes and its exact YAML header.
- **Responsibility**: the one job it owns.
- **Contract**: what its body must encode, step by step or field by field.
- **Edges**: what it spawns, loads, or depends on. Every skill named here is
  declared in the plugin's `dependencies`
  ([ADR-0006](../decisions/0006-declare-invoked-skills-as-dependencies.md)).

## Current-state discipline

A spec describes what is on disk **today**. A decided-but-unbuilt change is
flagged inline as *Pending ADR-NNNN* and specified in the ledger, never silently
folded in. That keeps the spec usable as a description and keeps the ledger the
only place a plan lives.

## Writing the bodies

The spec says what a body must encode. **How** to write it is the skill-writing
standard, which is a skill rather than a doc so it travels to other repos
([ADR-0009](../decisions/0009-own-the-skill-writing-standard.md)). The repo-local
house rules are in [`AGENTS.md`](../../AGENTS.md#house-rules).

## Index

| Plugin | Spec | Components |
|---|---|---|
| [`ship`](../../plugins/ship/) | [`ship.md`](ship.md) | `/ship`, `ship:implementer`, `ship:reviewer`, `/grill-pr`, `repo-contract` |
| [`tdd`](../../plugins/tdd/) | [`tdd.md`](tdd.md) | `/tdd` |
| [`committing`](../../plugins/committing/) | [`committing.md`](committing.md) | `/committing` |

## Planned (decided, not yet built)

A plugin that does not exist yet keeps its specification in the ledger, and gains
a file here when it is built.

- **Plugin `code-review`**, two skills sharing the review domain: a
  `review-baseline` library skill holding the review knowledge, and a
  `/code-review` wrapper that loads it for in-session use. Specified in full in
  [ADR-0010](../decisions/0010-extract-the-review-baseline-into-a-library-skill.md).
- The rest of the first pass is listed in [`PRD.md`](../PRD.md#first-pass).
