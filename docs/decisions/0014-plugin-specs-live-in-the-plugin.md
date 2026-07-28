# 0014. A plugin's spec lives in the plugin

- **Status:** Superseded by [ADR-0015](0015-specs-are-one-file-per-plugin-under-docs.md)
- **Date:** 2026-07-28

Amends [ADR-0012](0012-specify-per-component-not-per-repo.md), which made
`docs/COMPONENTS.md` canonical for every component contract. The per-component
bar it set is unchanged; only the location moves.

## Context and problem statement

`docs/COMPONENTS.md` held the spec for every component in the repo: five for
`ship`, one each for `tdd` and `committing`, at re-authoring depth. At three
plugins it was 350 lines, and the first pass in [`PRD.md`](../PRD.md) implies
roughly seven more.

The file was doing two unrelated jobs. One is a **contract**: what a spec must
contain, and the rule that nothing ships unspecified. That is repo-wide and
stable. The other is **the specs themselves**, which are per-plugin and change
whenever their plugin does. Only the second grows without bound, and it grows in
a file that sits far away from the thing it describes.

## Decision drivers

- [ADR-0011](0011-docs-must-reconstruct-the-repo.md) flagged this at the moment it
  created the file: "`COMPONENTS.md` restates what the prompts already encode. The
  two can drift." Distance is what makes drift easy.
- [`ARCHITECTURE.md`](../ARCHITECTURE.md) boundary 3 already argues the general
  case: agents ship inside the plugin that spawns them, because "the bundle is
  what stops a skill and its agent drifting apart." A spec is the same kind of
  thing as an agent brief in this respect.
- A spec beside its prompt is edited in the same commit and versioned by the same
  `version` bump. A spec in a distant file is edited by remembering to.
- [ADR-0009](0009-own-the-skill-writing-standard.md) already owns the portable
  half of "how to write a skill" and put it in a skill rather than a doc, so
  `COMPONENTS.md` must not absorb that when it slims down.
- A single overview has real value for an agent extending the system, and losing
  it entirely would be a cost.

## Considered options

- Specs move into `plugins/<name>/SPEC.md`; `COMPONENTS.md` keeps the contract
  and an index
- Specs move into the plugins; `COMPONENTS.md` is deleted and its rules fold into
  `AGENTS.md`
- Specs move into the plugins, but cross-cutting components stay central
- Leave `COMPONENTS.md` as it is

## Decision outcome

Chosen option: "Specs move into `plugins/<name>/SPEC.md`; `COMPONENTS.md` keeps
the contract and an index", because it puts each spec where it cannot drift
quietly while keeping one file that asserts no component ships unspecified.

Concretely:

- `plugins/<name>/SPEC.md` holds the plugin's own contracts and is **canonical
  for them**.
- `docs/COMPONENTS.md` holds the spec shape, the no-component-ships-unspecified
  rule, the current-state discipline, and an index of every plugin's spec. It is
  canonical for the **shape** of a contract, not for any component.
- The file is named `SPEC.md`, not `README.md`. Its audience is an agent
  re-authoring the component; the plugin's `description` and the root `README.md`
  serve someone browsing.
- A planned plugin has no directory, so its specification stays in the ledger
  until the directory exists, then moves in.

### Consequences

- Good, because a component and its spec share a directory, a commit, and a
  version bump, which is the cheapest drift protection available.
- Good, because `COMPONENTS.md` stops growing with the collection.
- Good, because a spec travels with its plugin, so a reader of one plugin has its
  contract to hand without the repo's doc set.
- Bad, because the single-file overview is gone. Reading every contract now means
  opening one file per plugin, and the index in `COMPONENTS.md` is what makes that
  navigable rather than pleasant.
- Bad, because `SPEC.md` is copied into the plugin cache on install, where its
  relative links to `../../docs/` dangle. Nobody reads a spec from the cache, so
  this is accepted rather than solved.
- Bad, because this is the second change to the doc set's shape in one sitting,
  after [ADR-0012](0012-specify-per-component-not-per-repo.md). The churn is real
  and both were reasoned rather than reflexive, but a third should be viewed with
  suspicion.

## Pros and cons of the options

### Specs in the plugin, `COMPONENTS.md` keeps the contract

- Good, because each half lives with what it is about.
- Good, because "no component ships unspecified" keeps a home that can assert it.
- Bad, because it is two files to look at instead of one.

### Delete `COMPONENTS.md` entirely

- Good, because it is the fewest files.
- Bad, because `AGENTS.md` is a contributor manual and `ARCHITECTURE.md` is a
  topology, so neither is a natural enforcer of the unspecified-component rule.
- Bad, because the index would have nowhere to live but a doc about something
  else.

### Cross-cutting components stay central

- Good, because a shared component would have one obvious home.
- Bad, because it creates two homes for one kind of content plus a rule about
  which applies.
- Bad, because nothing is actually cross-cutting yet: `repo-contract` lives inside
  `ship`, and `review-baseline` does not exist.

### Leave it as it is

- Good, because it changes nothing and every link still resolves.
- Bad, because it is the status quo whose growth prompted the question, and the
  drift risk was recorded against it from the day it was created.
