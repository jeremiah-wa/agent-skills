# 0013. Keep plugins in a `plugins/` directory

- **Status:** Accepted
- **Date:** 2026-07-28

## Context and problem statement

[`ARCHITECTURE.md`](../ARCHITECTURE.md) stated that the repo root holds one
directory per plugin. That was comfortable at one plugin. It reached three the
same day `tdd` and `committing` were authored, and the first pass in
[`PRD.md`](../PRD.md) implies roughly seven more, each interleaved at the root
with `docs/`, `README.md`, `AGENTS.md`, and three dot-directories.

A second idea was raised alongside it: a shared `skills/` tree that plugin
directories reference, so a skill exists once and several plugins point at it.
That turns out to be closed, and the two questions needed separating.

## Decision drivers

- Root-level concerns (the marketplace manifest, CI, the contributor contract,
  the doc set) should not be visually interleaved with content that grows without
  bound.
- The collection is private until switchover
  ([ADR-0007](0007-private-until-switchover.md)), so relative `source` paths are
  not yet cached by anyone. Moving directories is cheap exactly now and gets
  steadily more expensive.
- A nested `source` resolves: `"source": "./plugins/demo"` validates clean for
  both the marketplace manifest and the plugin manifest, confirmed by trial.
- A shared `skills/` tree does **not** resolve. `claude plugin validate` rejects a
  `..` segment in a plugin's `skills` array with "Path contains '..' which could
  be a path traversal attempt", which is stronger than
  [ADR-0003](0003-share-between-plugins-with-manifest-dependencies.md) recorded:
  it fails at authoring time, not only after install.
- Symlinks remain the only mechanism for real file sharing, and ADR-0003 already
  rejected them: they need elevation or Developer Mode on Windows, which is the
  primary development machine, and `--plugin-dir` and local-path installs skip
  them, so authoring and installed behaviour would differ.
- The `README.md` split into Workflows and Skills is described there as a reading
  aid. Making it a filesystem boundary would make it load-bearing.

## Considered options

- One `plugins/` directory holding every plugin
- Two directories, `workflows/` and `skills/`, mirroring the README taxonomy
- Leave every plugin at the repo root
- A shared `skills/` tree that plugin directories reference (not viable, listed
  for the record)

## Decision outcome

Chosen option: "One `plugins/` directory holding every plugin", because it is a
single rule with no judgement in it, and the alternative taxonomy would force a
category decision on every new plugin and a directory move whenever one changed
category.

Concretely:

- Every plugin is `plugins/<name>/`. `marketplace.json` sources become
  `./plugins/<name>`.
- The repo root holds only repo-level concerns.
- A plugin directory stays **self-contained**. There is no shared tree to draw
  from, and the validator enforces it.
- The CI workflow globs `plugins/*/` and now **fails when it finds no plugin
  manifests at all**, rather than passing green on an empty loop. This move would
  otherwise have produced exactly that silent pass.

### Consequences

- Good, because the root stays readable as the collection grows past ten plugins.
- Good, because it costs one rule and no per-plugin judgement.
- Good, because hardening the CI loop closed a real silent-green hole that had
  been there since the workflow was written.
- Bad, because every path reference in the doc set, the README install commands,
  and the authoring junctions on each machine had to change, and existing
  junctions must be repointed by hand or they keep resolving to nothing.
- Bad, because [ADR-0002](0002-one-plugin-per-unit-grouped-by-cohesion.md) and
  [ADR-0006](0006-declare-invoked-skills-as-dependencies.md) quote pre-move paths
  such as `ship/agents/implementer.md`. Records are append-only, so those stand as
  written and are read as historical.
- Neutral: plugin `name` is the install identity, not `source`, so nothing about
  dependency resolution or install naming changes.

## Pros and cons of the options

### One `plugins/` directory

- Good, because it is one rule, applied without judgement.
- Good, because a plugin never has to move once placed.
- Bad, because it adds a path segment to every reference.

### `workflows/` and `skills/`

- Good, because it mirrors a split the README already draws for readers.
- Bad, because the split is a reading aid, and the filesystem would make it
  binding.
- Bad, because `ship` is a workflow that also ships two skills, so the categories
  already overlap in the one case that exists.
- Bad, because a skill that grows an agent changes category and has to move.

### Leave plugins at the root

- Good, because it changes nothing and breaks no path.
- Bad, because it is the status quo that prompted the question, and it gets worse
  monotonically.

### A shared `skills/` tree

- Good, because a skill would exist once with no duplication.
- Bad, because the validator rejects `..` in a `skills` array outright.
- Bad, because the only remaining mechanism, symlinks, was already rejected by
  ADR-0003 on Windows and local-path-install grounds.
