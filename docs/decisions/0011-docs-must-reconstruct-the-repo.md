# 0011. The docs must be able to reconstruct the repo

- **Status:** Accepted
- **Date:** 2026-07-25

## Context and problem statement

[ADR-0008](0008-document-with-current-state-docs-and-a-ledger.md) gave the repo
current-state docs and a ledger, justified partly as giving the repo a spec so it
can serve as a test target. The mission (see [`PRD.md`](../PRD.md)) goes further:
the documented specification of the owner's practice is the artifact being kept,
and the skills are only its current compilation. That raises the bar on the docs.
How complete must they be?

## Decision drivers

- If the practice is the artifact, it cannot live only in the prompts. A prompt
  can be rewritten or rot; the specification has to survive it.
- The system is meant to be extended and rebuilt by agents, which need the
  specification at a definite standard, not prose that assumes tacit knowledge.
- Writing the practice down to the point of reconstructability is also what forces
  the owner to understand it: what cannot be specified is not yet understood.
- ADR-0008 already accepts a documentation-maintenance cost. Raising the bar
  raises that cost and duplicates content that also lives in the prompts.

## Considered options

- Docs sufficient to reconstruct the repo: add a component spec and a
  reconstruction guide, and hold a deletion test
- Thin current-state docs only, ADR-0008 as it stood: the prompts are the source,
  the docs describe them
- Generate the docs from the prompts, or the prompts from the docs, with tooling

## Decision outcome

Chosen option: "Docs sufficient to reconstruct the repo", because the mission
makes the specification the thing owned, and a specification you cannot rebuild
from is not the thing owned, only a description of it.

Concretely, on top of ADR-0008's set:

- `docs/COMPONENTS.md` specifies what every plugin and component must do, to
  re-authoring depth (frontmatter, responsibility, contract, edges).
- `docs/BUILD.md` is the reconstruction procedure: the file manifest, the root
  scaffolding reproduced verbatim, the `AGENTS.md` contract, and what
  "reconstructed" means.
- The bar is **behavioural and structural equivalence**, not byte-identity,
  because [ADR-0005](0005-author-from-a-blank-page.md) re-authors prose.
  Mechanical files (manifests, CI, `.gitignore`) are the exception and are
  reproduced verbatim.
- The check is a **deletion test**: remove everything but `docs/`, rebuild, and
  validate.

### Consequences

- Good, because the practice, not this year's prompts, is what is owned and
  portable.
- Good, because the reconstruction bar forces the practice to be fully
  articulated, which is the understanding driver.
- Good, because an agent can rebuild or extend the system from the spec alone.
- Bad, because `COMPONENTS.md` restates what the prompts already encode, and the
  `AGENTS.md` house rules now sit in two places. The two can drift.
- Mitigation: a component's spec is updated in the same change as the component,
  the coupling discipline ADR-0008 already relies on. The deletion test is the
  periodic proof that the discipline held.

## Pros and cons of the options

### Docs sufficient to reconstruct

- Good, because it makes the specification the artifact, matching the mission.
- Good, because it is testable: the deletion test either passes or it does not.
- Bad, because it is the most to write and the most to keep in sync.

### Thin docs only

- Good, because there is no duplication and less to maintain.
- Bad, because the practice then lives in the prompts, so a lost or rewritten
  prompt loses part of the practice with no record.

### Generate one from the other

- Good, because a single source removes the drift.
- Bad, because there is no such tool here, the content is prose both ways, and a
  generator would encode its own opinions about the practice, which is the
  opposite of owning it.
