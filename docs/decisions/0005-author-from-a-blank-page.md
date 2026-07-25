# 0005. Author skills from a blank page, treating prior art as reference

- **Status:** Accepted
- **Date:** 2026-07-25

## Context and problem statement

[ADR-0004](0004-own-the-skills-we-use.md) settled that we write our own versions
of the skills we use. It did not settle *how* a skill gets written, and the
options differ sharply in speed, in voice, and in legal obligation.

## Decision drivers

- The stated intent is that these skills are the author's own, encoding the
  author's practice rather than someone else's.
- Copyright covers **expression**, not ideas or terminology. The established
  vocabulary (progressive disclosure, information hierarchy, leading word, no-op,
  sediment, premature completion) is technical language, freely usable.
- Copy-and-edit produces a **derivative work**. MIT then obliges carrying the
  upstream copyright and permission notice.
- A file that starts as someone else's keeps their structure and voice long after
  the words change.
- Writing from scratch is slow, and slowness is a filter: it forces the question
  of whether this skill is one we actually have an opinion about.

## Considered options

- Blank page, prior art as reference
- Copy then edit
- Mixed, decided per skill

## Decision outcome

Chosen option: "Blank page, prior art as reference", because the goal is
authorship, and a derivative file is not authored.

Read the prior art, close it, write ours. Keep the established vocabulary; write
original prose and original opinions.

This is how a skill enters the repo at every stage, not only at the bootstrap.
Self-invention is the phase, not authorship: later, an external skill may be
adopted rather than invented, but adoption runs through the same blank page. It
qualifies only once it has earned its place by **examined use** (the author has
used it and judged it good), and it is then written here to the house standard and
voice, never copied. A conformed adoption is therefore an authored skill carrying
no attribution obligation, which is exactly why the blank-page rule holds whether
the idea was invented or found.

### Consequences

- Good, because no attribution obligation attaches to anything in this repo.
- Good, because the result is in the author's voice and structure from line one.
- Good, because it forces a per-skill judgement about whether the skill is wanted.
- Bad, because it is slower per skill, and upstream refinement is not inherited.
- Note: crediting prior art in `README.md` is courtesy rather than obligation, and
  is worth doing.

## Pros and cons of the options

### Blank page

- Good, because the output is genuinely original and owes nothing.
- Good, because vocabulary is still shared, so nothing has to be reinvented.
- Bad, because it is the slowest route and loses upstream polish.

### Copy then edit

- Good, because it is fast and inherits refinement already paid for.
- Bad, because it is a derivative work carrying an MIT notice obligation.
- Bad, because the inherited structure and voice persist well past the rewrite.

### Mixed per skill

- Good, because it is pragmatic: copy where the original is close, write where it
  is not.
- Bad, because which files are derivative must then be tracked for attribution,
  and that thread is easy to lose.
