# agent-skills, Product Requirements Document

**Version:** 0.1
**Date:** 2026-07-25
**Status:** Draft, in progress

> **Temporary document.** This exists to answer "what belongs in this repo and
> what does not" while that is still being settled. `README.md` now exists and
> carries the catalog, so what migrates when this settles is the **mission**, not
> the file; the rest goes away with it.

---

## Mission

This repo is the documented specification of how I engineer. The skills and
workflows are the current compilation of that practice; the specification is the
thing being kept.

That inverts the usual order. A collection of skills is normally the artifact,
and docs describe it. Here the docs are the artifact, and the skills are one
rendering of them, the way the practice happens to execute this year. Anything
that lives only in a prompt is not yet specified, and specifying it is the
point.

The bar is set per component rather than per repo: every plugin, skill, and
agent is described to the level another agent needs to re-author it from a blank
page ([ADR-0012](decisions/0012-specify-per-component-not-per-repo.md)). What
that rules out is a doc that only makes sense to someone who has already read
the prompt it describes.

One test survives, and unlike the old one it is cheap enough to actually run:
**a step of the practice that works only on the machine it was written on is not
yet owned.** [`PRACTICE.md`](PRACTICE.md) maps every step of every workflow to
the skill that serves it and marks whether that skill is in this repo. Today,
two nodes of the trunk workflow are.

Three drives, one spine:

- **Own the practice, do not inherit it.** Every skill is authored from a blank
  page so the judgement in it is mine, not absorbed wholesale from someone
  else's collection. The enemy is unexamined inheritance, not external ideas
  ([ADR-0004](decisions/0004-own-the-skills-we-use.md),
  [ADR-0005](decisions/0005-author-from-a-blank-page.md)).
- **Write it down to understand it.** Forcing the practice onto the page is how
  I keep understanding it and keep it coherent. A practice that lives only in
  the hands cannot be corrected, only repeated.
- **Grow it with agents.** The system is meant to be extended, refactored, and
  rebuilt by agents, so it is specified to the standard an agent needs to extend
  it faithfully
  ([ADR-0012](decisions/0012-specify-per-component-not-per-repo.md)).

Self-invention is the phase, not ownership. Every skill is authored here from a
blank page, at the bootstrap and after. What changes is where the ideas come
from: at first my own practice, later also external skills I have used and
judged good. Two gates hold at every stage: **examined use** (nothing enters
that I have not used and judged) and **conformance** (nothing enters as a
derivative copy; prior art is re-authored to the house standard and voice, per
[ADR-0005](decisions/0005-author-from-a-blank-page.md)). One invariant never
moves: every skill here is one I have an opinion about.

## Problem Statement

From the perspective of a solo engineer working across several repos and more
than one machine:

- The agent skills and workflows that make the work faster live in scattered,
  machine-local places. Some are symlinks from a third party's collection, some
  are hand-written directories that exist on exactly **one** computer, and
  nothing records which is which.
- A workflow assembled from those pieces looks portable and is not. `ship` names
  two skills it does not ship and cannot check for, so it worked on the machine
  it was written on and nowhere else. Neither install channel reported a
  problem.
- The skills that shape the work are **someone else's opinions**. They are good
  opinions, but they encode how their author works, and using them means
  inheriting choices rather than making them.
- There is no single command that puts a known-good set of tools on a new
  machine.

## Solution

**agent-skills** is a personal collection of Claude Code skills and workflows,
authored by and for me, installable on any machine in one command.

It is:

- **Owned.** Every skill here is written from a blank page. Third-party
  collections are read as prior art and closed. Established vocabulary carries
  over; expression does not.
- **Self-declaring.** Every skill a workflow invokes is declared as a plugin
  dependency, so a machine missing something says so at install time instead of
  degrading quietly at runtime.
- **Packaged by cohesion.** Things that share files and change together ship
  together. Everything else ships alone and can be installed on its own.
- **Self-specifying.** Every component is described to the level another agent
  needs to re-author it from a blank page. That also makes the repo a subject
  for the workflows it contains, not just their source
  ([ADR-0012](decisions/0012-specify-per-component-not-per-repo.md)).

## In scope

- Skills I **actually use**, evidenced by usage rather than assumed. The counts
  and the full never-invoked list live in [`PRACTICE.md`](PRACTICE.md#evidence).
  In short, across 127 transcripts: `grill-me` and `grilling` (25 invocations
  between them), `implement` (8), `to-issues` and `to-prd` (5), `code-review`
  (3), `committing` (2), then a long tail of one.
- Workflows that coordinate agents, currently `ship`.
- A house standard for writing skills, owned here rather than borrowed.
- A docs skill owning this repo's own doc conventions (glossary, ADRs, PRD,
  architecture, component specs, and the reconstruction guide).

## Out of scope

- **Skills nobody uses.** 14 of the 24 skills in the third-party collection this
  replaces have never been invoked once, including its four largest. They are
  not reproduced here.
- **Other agent harnesses.** Claude Code only. Cursor, Codex, and Amp are not
  targets, and the distribution channel was chosen on that basis
  ([ADR-0001](decisions/0001-distribute-as-claude-code-plugins.md)).
- **Republishing third-party work.** Nothing is copied in verbatim from another
  collection. An external skill is adopted later only by re-authoring it to the
  house standard once it has earned its place by use, never by redistributing
  someone else's expression.
- **General-purpose or shareable tooling.** This is one person's practice. It
  may become public ([ADR-0007](decisions/0007-private-until-switchover.md)),
  but it is not written for an audience.

## First pass

The set to write before the collection can stand on its own:

| Skill | Why |
|---|---|
| `grilling` + `grill-me` | Most-used by more than 2x, and small |
| ~~`tdd` + `committing`~~ | **Done.** What `ship`'s implementer invokes. Writing these is what made `ship` installable anywhere |
| `implement` | Second most-used |
| `code-review` | In use; overlap with `ship:reviewer` settled by [ADR-0010](decisions/0010-extract-the-review-baseline-into-a-library-skill.md): a `review-baseline` library skill both load |
| `grill-with-docs` | Grilling that produces docs, composed with the docs skill |
| docs skill | Owns glossary, ADR, PRD, and architecture conventions, modelled on `plain-sight` |
| skill-writing standard | Thin, grown over time ([ADR-0009](decisions/0009-own-the-skill-writing-standard.md)) |

`ship` now declares `dependencies: ["tdd", "committing"]`, and gains
`code-review` once that plugin exists
([ADR-0010](decisions/0010-extract-the-review-baseline-into-a-library-skill.md)).

## Non-goals for v1

- A bundle plugin for one-command install of everything. Undecided, and
  premature at three plugins.
- Version constraints on dependencies. Bare names until a real compatibility
  break appears, since constraints oblige per-plugin git tags.
- Extracting shared content into its own plugin. Nothing is extracted until a
  second consumer exists.

## Open questions

1. **`implement` vs `ship`.** Both take work to code. `implement` runs
   in-session on the current branch with no isolation; `ship` runs an autonomous
   worktree-and-PR loop. Same task at opposite ends of a ceremony dial. Do both
   exist?
2. ~~**`code-review` vs `ship:reviewer`.**~~ **Resolved by
   [ADR-0010](decisions/0010-extract-the-review-baseline-into-a-library-skill.md).**
   The baseline lives once, in a `review-baseline` library skill inside a new
   `code-review` plugin. `ship:reviewer` gains the `Skill` tool and loads it; a
   standalone `/code-review` wraps it for in-session use and exists alongside
   `grill-pr`.
3. **A bundle plugin**, whether and what it is called.
4. **Names** for the docs skill and the `implement` equivalent.

## Success

A new machine runs two commands and has the working set. Nothing in this repo
resolves by accident of what happens to be installed. Every skill here is one
its I have an opinion about. And the specification holds: delete everything but
`docs/`, and an agent rebuilds the repo from it.
