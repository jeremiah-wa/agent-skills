# 0012. Specify per component, not per repo

- **Status:** Accepted
- **Date:** 2026-07-28

Supersedes [ADR-0011](0011-docs-must-reconstruct-the-repo.md).

## Context and problem statement

[ADR-0011](0011-docs-must-reconstruct-the-repo.md) set the documentation bar as a
**deletion test**: remove everything but `docs/`, rebuild from `docs/BUILD.md`,
and validate. To survive that deletion, `BUILD.md` had to carry verbatim copies
of every file outside `docs/`, including the root `README.md` and `AGENTS.md`.

Three months of living with it, the costs are concrete and the benefit is not.
`BUILD.md:246-248` states outright that the contributor contract is
"single-sourced here so it survives the deletion test", and the result is a repo
with **no `AGENTS.md` and no `README.md` on disk at all**, both existing only as
templates inside a build procedure. Meanwhile the test has never been run, and
realistically will not be: it requires an agent to rebuild the entire repo and a
human to judge behavioural equivalence.

The bind is not escapable by tidying. "Delete everything but `docs/` and rebuild"
and "do not copy file content into `docs/`" are in direct conflict. If the
mechanical files are deleted by the test, `docs/` must contain them.

## Decision drivers

- A test that is never run is not a check, it is an aspiration, and it was
  enforcing exactly what discipline alone would enforce.
- The test decided where the contributor contract lives, which is the tail
  wagging the dog. `AGENTS.md` is the file agents load when working here; it
  should be a file.
- The valuable half of ADR-0011 is per-component: `COMPONENTS.md` is good
  *because* someone had to be able to re-author from it, and `COMPONENTS.md:3-4`
  already states that bar in its own words, independent of any deletion test.
- The mission in [`PRD.md`](../PRD.md) cited the deletion test as literal, so this
  is a mission edit, not only a doc deletion.
- A cheaper test became available: `docs/PRACTICE.md` marks every step of the
  practice with whether the skill serving it is in this repo. "Does this work on a
  machine that is not this one" is answerable in seconds and has a real answer
  today.

## Considered options

- Keep the deletion test and accept the duplication
- Specify per component; mechanical files are their own source
- Keep a reconstruction test but scope it to one plugin at a time
- Drop the reconstruction idea entirely with nothing in its place

## Decision outcome

Chosen option: "Specify per component; mechanical files are their own source",
because the per-component bar is what actually forced the docs to be good, and
the repo-wide deletion test was paying for it with duplication and two missing
root files.

Concretely:

- **`docs/BUILD.md` is deleted.** Its recreation procedure, file manifest, and
  verbatim scaffolding all existed to serve the deletion test.
- **`README.md` and `AGENTS.md` become real files at the repo root**, promoted
  from `BUILD.md`'s templates. They are now canonical, not copies.
- **`AGENTS.md` gains the doc-update rules**: which doc owns what, and what change
  obliges an edit to which file. That is what now enforces currency, replacing the
  deletion test.
- **The bar is per component.** Every plugin, skill, and agent is specified in
  `COMPONENTS.md` to the level another agent needs to re-author it from a blank
  page. That is unchanged from ADR-0011 and is the part worth keeping.
- **Mechanical files are their own source.** Manifests, the CI workflow, and
  `.gitignore` are not restated in `docs/`. The plugin manifest template moves to
  `AGENTS.md`, where scaffolding a new plugin is a contributor concern.
- **Diagrams and prose in `docs/` are illustrative unless a doc says otherwise.**
  `COMPONENTS.md` is normative for component contracts. `PRACTICE.md` is
  illustrative and says so.

The portability check in `PRACTICE.md` replaces the deletion test as the mission's
stated test. It is weaker on completeness and stronger on being run.

### Consequences

- Good, because the repo gets an `AGENTS.md` and a `README.md`, which it has
  never had.
- Good, because the scaffolding duplication goes away: four of the six files
  `BUILD.md` copied already existed on disk.
- Good, because the remaining test is cheap enough to actually run, and gets more
  informative as skills migrate into the repo.
- Bad, because the completeness forcing function is now an assertion in
  `COMPONENTS.md` with nothing mechanical behind it. Doc currency rests entirely
  on the `AGENTS.md` rules and on discipline, which
  [ADR-0008](0008-document-with-current-state-docs-and-a-ledger.md) already
  flagged as the weak link.
- Bad, because the mission paragraph in `PRD.md` must be rewritten, and a claim
  the repo made about itself is being walked back rather than met.
- Neutral: nothing about `ADR-0005` blank-page authoring changes. Two bodies
  satisfying one contract are still both correct.

## Pros and cons of the options

### Keep the deletion test

- Good, because the bar is unambiguous and testable in principle.
- Good, because it forced `COMPONENTS.md` to a standard it would not otherwise
  have reached.
- Bad, because it obliges `docs/` to carry copies of files that exist on disk.
- Bad, because it left the repo with no `AGENTS.md` and no `README.md`.
- Bad, because it was never run once.

### Specify per component

- Good, because it keeps the half of ADR-0011 that produced the value.
- Good, because every file lives where it belongs and is its own source.
- Bad, because there is no longer any check, even a theoretical one, on whether
  `docs/` is complete.

### Per-plugin reconstruction test

- Good, because it is small enough to actually run: rebuild one plugin from its
  `COMPONENTS.md` spec and compare behaviour.
- Bad, because it still needs the plugin's mechanical files specified somewhere,
  reintroducing a smaller version of the same duplication.
- Bad, because it adds a ceremony to every plugin at a point where the repo has
  one plugin and is trying to grow.

### Drop reconstruction entirely

- Good, because it is the least to maintain.
- Bad, because it removes the reason `COMPONENTS.md` is written the way it is,
  and that doc is the repo's best asset.
