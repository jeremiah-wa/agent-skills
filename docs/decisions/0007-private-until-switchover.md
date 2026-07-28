# 0007. Keep the repository private until switchover

- **Status:** Accepted
- **Date:** 2026-07-25

## Context and problem statement

At the time of this decision the repo had no commits and no remote, so neither
documented install path resolved. Pushing it is the actual unblocker for
"installable on any machine". That makes visibility a decision rather than a
default.

## Decision drivers

- A public marketplace source needs **no git credentials**, which is the
  shortest path to "any machine".
- A private one still installs, but every machine needs credentials for the repo
  (`gh auth` or a credential helper) before the marketplace resolves.
- The collection is unfinished. Most of the skills named in
  [ADR-0004](0004-own-the-skills-we-use.md) do not exist yet.
- The period before switchover is the period of **most active syncing** between
  machines, which is exactly when credential friction costs most.

## Considered options

- Public now
- Private permanently
- Private now, public at switchover

## Decision outcome

Chosen option: "Private now, public at switchover", tied to the same judgement
call that retires `mattpocock-skills`.

### Consequences

- Good, because a half-written collection is not public while it is
  half-written.
- Good, because the trigger is one I already have to make, rather than a second
  thing to remember.
- Bad, because the auth friction lands during the period of heaviest
  cross-machine use, which is the cost this decision knowingly accepts.

## Pros and cons of the options

### Public now

- Good, because install is credential-free everywhere immediately.
- Good, because prior art can be credited visibly.
- Bad, because unfinished work and working practice are public while in
  progress.

### Private permanently

- Good, because nothing is ever exposed.
- Bad, because it permanently keeps friction on the path this repo exists to
  make frictionless.

### Private now, public at switchover

- Good, because it defers exposure without deferring it forever.
- Bad, because it carries the friction through the busiest period.
