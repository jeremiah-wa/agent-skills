# 0009. Own the skill-writing standard as a thin skill

- **Status:** Accepted
- **Date:** 2026-07-25

## Context and problem statement

`AGENTS.md` defers the house writing standard to `writing-great-skills`, a
third-party skill that [ADR-0004](0004-own-the-skills-we-use.md) retires. That
leaves the standard homeless at switchover, and the standard is what every other
skill in this repo will be written against.

## Decision drivers

- The standard applies **wherever a skill is written**, not only in this repo, so
  it belongs in a skill rather than in `AGENTS.md`.
- The prior art is a **theory**, not a procedure: roughly 28KB across an 18KB
  glossary defining about 25 terms on four axes, plus a 9KB skill. It is the
  largest single item on the first-pass list by a wide margin.
- Writing that theory first would block every other skill behind it, while the
  third-party original still works.
- Vocabulary is freely usable ([ADR-0005](0005-author-from-a-blank-page.md)), so
  a thin standard can lean on established terms without defining them from
  scratch.
- Several house rules are already written and already applied (prompt the
  positive, every line earns its place, prune on every edit, disclose reference
  behind pointers). The standard starts non-empty.

## Considered options

- Thin standard now, grown over time
- Write the full theory first
- Expand `AGENTS.md`, no skill
- Defer entirely until switchover

## Decision outcome

Chosen option: "Thin standard now, grown over time". Write a one-page house
standard as our own skill immediately, capturing the rules already applied.
`AGENTS.md` defers to it instead of to `writing-great-skills`. It grows a section
each time something needs re-explaining.

### Consequences

- Good, because it unblocks the rest of the first pass today rather than gating it
  behind a theory.
- Good, because it is portable, usable when writing skills in any repo.
- Good, because it grows from observed need rather than from speculation, which is
  the same discipline it will preach.
- Bad, because thin means it will miss things early, and skills written in that
  window may need revisiting.

## Pros and cons of the options

### Thin standard now

- Good, because it is written from rules already in use, so it starts true.
- Bad, because it is incomplete by construction.

### Full theory first

- Good, because everything downstream is written to a settled standard from line
  one.
- Bad, because it is the biggest item on the list, and the third-party original
  still works in the meantime.

### Expand `AGENTS.md`

- Good, because it is one file and needs no new plugin.
- Bad, because the guidance is then trapped in this repo and unavailable when
  writing a skill elsewhere.
- Bad, because `AGENTS.md` stops being thin.

### Defer entirely

- Good, because zero work today.
- Bad, because switchover gains a hard blocker, and everything written before then
  is written to someone else's standard.
