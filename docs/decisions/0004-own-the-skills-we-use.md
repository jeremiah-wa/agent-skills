# 0004. Own the skills we use rather than depend on a third-party collection

- **Status:** Accepted
- **Date:** 2026-07-25

## Context and problem statement

My daily skills come from
[mattpocock/skills](https://github.com/mattpocock/skills), MIT licensed,
distributed as a single plugin (`mattpocock-skills`) containing 22 skills.
`ship`'s implementer already calls `Skill(tdd)` from it. The intent for this repo
is a personal collection expressing how I work, which raises the question of
what relationship this repo should have with that one.

## Decision drivers

- The stated intent is ownership. These skills should be mine, in my voice,
  encoding my practice.
- `mattpocock-skills` is **one plugin**, so a single skill cannot be retired
  from it. It is taken whole or not at all.
- Two skills with the same name, one from a plugin and one from
  `~/.claude/skills/`, have no documented precedence.
- Usage evidence across 98 transcripts: `grilling` and `grill-me` account for 17
  invocations, `implement` 8, `code-review` 3, `tdd` 1, `writing-great-skills`
  1, `handoff` 1. **14 of the 24 skills have never been invoked**, including the
  four largest.
- A fork of an actively developed third-party collection (roughly 3,100 lines
  across 24 skills, released with changesets and version tags) is a standing
  maintenance liability.

## Considered options

- Write our own versions of the skills actually used
- Cross-marketplace dependency on `mattpocock-skills`
- Vendor only the two skills `ship` needs, keep depending on the rest
- Fork the whole collection

## Decision outcome

Chosen option: "Write our own versions of the skills actually used", because the
goal is authorship rather than availability, and the usage data shows the set
that actually matters is about six skills totalling under 400 lines.

First pass: `grilling`, `grill-me`, `tdd`, `committing`, `implement`,
`code-review`, `grill-with-docs`, a docs skill, and a skill-writing standard
(see [ADR-0009](0009-own-the-skill-writing-standard.md)).

**Switchover has no fixed trigger.** `mattpocock-skills` stays installed until
I judge this repo ready.

### Consequences

- Good, because there is no third-party dependency, no cross-marketplace
  allowlist, and no fork to maintain.
- Good, because the scope is far smaller than the collection's size suggests.
- Bad, because it is real writing work before switchover, and
  `writing-great-skills` in particular is a substantial theory rather than a
  procedure.
- Note on the interim: `mattpocock-skills` is installed by the `skills` CLI as
  **symlinks in `~/.claude/skills/<name>`**, the same path the authoring
  junctions use. Writing our own therefore **replaces** his at that path rather
  than duplicating it. This only holds while his install stays on that CLI;
  moving him to a marketplace plugin would make both coexist with undocumented
  precedence. Leave him on the CLI until switchover.
- Note: `committing` was found to exist only as a local directory in
  `~/.claude/skills/`, unversioned and on one machine. See
  [ADR-0006](0006-declare-invoked-skills-as-dependencies.md).

## Pros and cons of the options

### Write our own

- Good, because the result is genuinely mine, which is the point.
- Good, because only skills I have an opinion about get written.
- Bad, because upstream improvements are not inherited.

### Cross-marketplace dependency

- Good, because it is one line of config and upstream updates keep flowing.
- Good, because it creates no duplicate names.
- Bad, because it makes the collection depend permanently on someone else's
  repo.
- Bad, because it pulls all 22 skills, 14 of which have never been used.
- Bad, because a version constraint would fail: upstream tags are `v1.1.0`, not
  the `mattpocock-skills--v1.1.0` convention resolution requires.

### Vendor only what `ship` needs

- Good, because it makes `ship` self-contained for the least work.
- Bad, because a vendored `tdd` duplicates his for as long as his plugin stays
  installed, and it cannot be disabled out of his bag.

### Fork the whole collection

- Good, because total control, offline, nothing external.
- Bad, because it is roughly 3,100 lines of someone else's actively developed
  work to maintain.
- Bad, because parts of it make no sense outside his authorship (`ask-matt`,
  `setup-matt-pocock-skills`), and vendoring a subset dangles the
  cross-references that skills make to their siblings.
