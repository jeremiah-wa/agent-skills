# 0008. Document the repo with current-state docs and an append-only ledger

- **Status:** Accepted
- **Date:** 2026-07-25

## Context and problem statement

The repo's conventions lived only in `AGENTS.md`, which asserted four invariants
with no recorded reasons. One of them turned out to be wrong (see
[ADR-0003](0003-share-between-plugins-with-manifest-dependencies.md)), and
nothing in the repo could have caught it, because nothing recorded why it was
written. The sibling repo `plain-sight` already runs a working doc system worth
copying.

## Decision drivers

- The perishable part of a decision is its **drivers and rejected options**. The
  outcome survives in the code; the reasoning does not survive anywhere.
- Vocabulary ambiguity caused real confusion during design: one proposed name
  covered two opposite roles (a shared library that everything depends on, and a
  bundle that depends on everything) and the conflict stayed invisible until the
  dependency direction was questioned directly.
- `AGENTS.md` states its own design as "thin by design, defers rather than
  restates", so it should shed anything a dedicated doc can own.
- This repo is also intended as a **test target for the author's own agentic
  workflow**, and a repo with no documented spec is a poor target.

## Considered options

- `AGENTS.md` only
- ADRs only
- ADRs plus a glossary
- Current-state docs plus an append-only ledger (the `plain-sight` shape)

## Decision outcome

Chosen option: "Current-state docs plus an append-only ledger", following
`plain-sight`: `PRD.md` and `ARCHITECTURE.md` describe the current state, and
`docs/decisions/` is the immutable dated account of what changed it.

- `docs/PRD.md` answers what belongs in this repo and what does not. It is
  **temporary** and becomes `README.md` once it settles.
- `docs/ARCHITECTURE.md` describes the plugin topology and how things resolve at
  install and at runtime.
- `docs/GLOSSARY.md` pins the vocabulary.
- `docs/decisions/` holds MADR records, append-only.

`AGENTS.md` stays thin and defers to these rather than restating them.

[ADR-0011](0011-docs-must-reconstruct-the-repo.md) later raises this doc set from
a description to a reconstructable specification, adding `docs/COMPONENTS.md` and
`docs/BUILD.md`, once the mission made the specification the artifact being kept
rather than a support for the code.

### Consequences

- Good, because reasons are recorded where they can be corrected once instead of
  re-guessed every time.
- Good, because rejected options are captured while they are still live, which is
  the half of MADR that cannot be reconstructed later.
- Good, because it gives this repo a spec, which it needs to serve as a test
  target for its own workflows.
- Bad, because four docs must be kept current, and the coupling rule (an ADR also
  edits the canonical doc and links back) is enforced by discipline alone.
- Bad, because this repo remains a **weak test of `ship`'s verify loop**: there is
  no build and no test suite, so `claude plugin validate --strict` is the only
  mechanical gate. It exercises the orchestration (worktrees, handoff,
  dispositions, breakers) and barely exercises the parts that depend on CI.

## Pros and cons of the options

### `AGENTS.md` only

- Good, because it is one file, already loaded by agents working here.
- Bad, because it is the status quo that produced a wrong invariant with no way to
  catch it.
- Bad, because it would stop being thin, which is its stated design.

### ADRs only

- Good, because it captures the perishable part at minimum cost.
- Bad, because the vocabulary stays unpinned, and vocabulary was demonstrably
  ambiguous.

### ADRs plus a glossary

- Good, because it covers both problems actually observed.
- Bad, because the current-state description then has no home outside `AGENTS.md`.

### Current-state docs plus a ledger

- Good, because it separates what is true now from why it changed.
- Bad, because it is the most to write and the most to keep current.
