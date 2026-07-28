# 0015. Specs are one file per plugin, under `docs/specs/`

- **Status:** Accepted
- **Date:** 2026-07-28

Supersedes [ADR-0014](0014-plugin-specs-live-in-the-plugin.md). Further amends
[ADR-0012](0012-specify-per-component-not-per-repo.md) on location only; the
per-component bar both set is unchanged.

## Context and problem statement

Location has now moved twice in a day. ADR-0012 put every component spec in one
`docs/COMPONENTS.md`. ADR-0014 moved the specs into `plugins/<name>/SPEC.md`, on
the argument that a spec beside its prompt cannot drift quietly. Both left a
real problem standing.

`COMPONENTS.md` does not scale: 350 lines at three plugins, with roughly seven
more implied by the first pass in [`PRD.md`](../PRD.md). And a spec inside a
plugin ships to consumers, because the plugin directory is copied wholesale into
the plugin cache on install.

## Decision drivers

- **The plugin directory is the shipped artifact.** A specification is internal
  to authoring this collection and is not part of what someone installs. Keeping
  the shipped thing to what it needs is worth something on its own.
- The repo has not shipped, so relocating costs nothing but edits. This is the
  cheapest moment it will ever be.
- The context argument for moving specs out of plugins does **not** hold and is
  recorded here so it is not reused: Claude Code auto-loads `SKILL.md` and agent
  definitions, not arbitrary markdown in a plugin directory. A `SPEC.md` there
  costs a few KB of cache and no context at all.
- One file per plugin scales where one file per repo does not, and the plugin is
  already the unit of install and versioning
  ([`GLOSSARY.md`](../GLOSSARY.md)), which is what a spec's `version` line tracks.
- This repo already solved the same shape of problem in `docs/decisions/`: many
  records, one directory, a `README.md` holding the index and the format rules.
  A proven local pattern beats a new one.

## Considered options

- One file per plugin under `docs/specs/`, with `README.md` holding the contract
  and index
- Keep specs in `plugins/<name>/SPEC.md` (ADR-0014, in force until now)
- Keep one `docs/COMPONENTS.md` (ADR-0012)
- One file per component under `docs/specs/<plugin>/`

## Decision outcome

Chosen option: "One file per plugin under `docs/specs/`", because it keeps the
shipped artifact clean and scales with the collection, using a directory pattern
the repo already runs successfully for its ledger.

Concretely:

- `docs/specs/<plugin>.md` holds that plugin's contracts and is canonical for
  them.
- `docs/specs/README.md` holds the spec shape, the
  no-component-ships-unspecified rule, the current-state discipline, and the
  index. It is canonical for the **shape** of a contract, not for any component.
- `docs/COMPONENTS.md` is deleted; its content became `docs/specs/README.md`.
- `plugins/<name>/SPEC.md` is removed. A plugin directory holds only what it
  ships.
- A planned plugin keeps its specification in the ledger and gains a file here
  when it is built.

### Consequences

- Good, because a plugin directory now contains only what a consumer installs.
- Good, because specs scale one file at a time instead of growing one file
  without bound.
- Good, because `docs/specs/` and `docs/decisions/` are the same shape, so there
  is one pattern to learn rather than two.
- Bad, because ADR-0014's drift protection is given up. A component and its spec
  are in different trees again, and only the trigger table in
  [`AGENTS.md`](../../AGENTS.md) holds them together. That was a real benefit and
  this trades it away deliberately.
- Bad, because location has now changed three times in one day. The ledger shows
  it rather than hiding it, which is the point of an append-only record, but the
  next proposal to move specs should carry a much higher burden.

## Pros and cons of the options

### One file per plugin under `docs/specs/`

- Good, because it scales and keeps the shipped artifact clean.
- Good, because it reuses the `docs/decisions/` pattern.
- Bad, because spec and component sit in different trees and can drift.

### Specs in `plugins/<name>/SPEC.md`

- Good, because a component and its spec share a directory, a commit, and a
  version bump, which is the strongest drift protection available.
- Good, because a spec travels with its plugin.
- Bad, because it ships internal specification to everyone who installs.
- Bad, because its relative links to `docs/` dangle once copied into the cache.

### One `docs/COMPONENTS.md`

- Good, because everything is readable in one pass.
- Bad, because it was already 350 lines at three plugins and grows with every
  addition.

### One file per component

- Good, because each file has exactly one subject.
- Bad, because `ship`'s five components reference each other constantly and the
  orchestrator's contract is meaningless without the agents it spawns.
- Bad, because it would be fifteen files where three do the job.
