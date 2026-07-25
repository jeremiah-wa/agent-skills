# 0010. Extract the review baseline into a shared library skill

- **Status:** Accepted
- **Date:** 2026-07-25

## Context and problem statement

The adversarial review baseline (the "assume it's broken" stance, the twelve-item
smell list, and the hunt then verify then failure-scenario discipline) lives
inline in `ship/agents/reviewer.md`. [ADR-0006](0006-declare-invoked-skills-as-dependencies.md)
noted this material would be duplicated the moment a standalone `code-review`
skill existed, and left it as an open follow-up. `PRD.md` open-question #2 asks
the same thing: where does the baseline live, and does a standalone `code-review`
exist alongside `grill-pr`? This record answers both. Implementation is pending;
this decides the shape.

## Decision drivers

- The baseline is **knowledge**. A reviewer's coldness, worktree isolation, and
  `gh pr review` emission are **execution**. An ad hoc in-session review and
  `ship`'s cold PR review run in different execution contexts, so only the
  knowledge is ever shared between them.
- Two consumers embed the baseline independently: `ship:reviewer` and a future
  standalone review. `grill-pr` spawns `ship:reviewer` and embeds nothing.
- Cross-plugin composition works by naming a skill in prose; a dependency grants
  reachability of named components, not files
  ([ADR-0003](0003-share-between-plugins-with-manifest-dependencies.md),
  [ADR-0006](0006-declare-invoked-skills-as-dependencies.md)). A cold agent may
  fetch what it needs itself (`AGENTS.md`), so `Skill(review-baseline)` does not
  break the cold-agent invariant.
- `ship:reviewer`'s `tools` list omits `Skill`, the exact reason ADR-0006 gave
  for keeping the baseline inline.
- A dependency cycle is forbidden: `grill-pr` spawns `ship:reviewer`, so it
  cannot move out of `ship` into a plugin that `ship` depends on.

## Considered options

- Extract the baseline to a `review-baseline` library skill, give `ship:reviewer`
  the `Skill` tool, and have both reviewers load it (Packaging 1)
- Make the standalone `code-review` skill itself the baseline and have the
  reviewer call it directly (Packaging 2)
- Invert control: a `code-review` plugin owns the reviewer agent and `ship`
  spawns it
- Do nothing, and accept the copy when a standalone appears

## Decision outcome

Chosen option: "Extract the baseline to a `review-baseline` library skill",
because it puts the shared knowledge in exactly one place while letting each
reviewer keep its own execution, at the cost only of adding the `Skill` tool to
the reviewer, a boundary ADR-0006 documented but did not defend.

Concretely:

- A new `code-review` plugin holds two skills: `skills/review-baseline/SKILL.md`
  (the knowledge, `disable-model-invocation: true`) and `SKILL.md`
  (`/code-review`, an in-session wrapper that loads it). They share the review
  domain and change together, so one plugin by the cohesion rule
  ([ADR-0002](0002-one-plugin-per-unit-grouped-by-cohesion.md)).
- `/code-review` reviews the working-tree diff by default, falls back to the
  merge-base with the default branch, and accepts an explicit base. With no spec
  it skips the spec axis, as `grill-pr` does.
- `ship/agents/reviewer.md` gains `Skill` in `tools`, drops the inline baseline,
  and calls `Skill(review-baseline)`. Its PR execution shell (ground via
  `gh pr`, post `gh pr review`) stays.
- `ship` declares `dependencies: ["tdd", "committing", "code-review"]`.
- `grill-pr` stays in `ship`; moving it would cycle. It needs no change.
- Two lines are execution, not knowledge, and stay in the wrappers: the "CI is
  green, so lint, format, and types are not yours to report" assumption (true
  only for `ship`, which gates on CI), and the request-changes-or-approve verdict
  (a `gh pr review` act; in-session there is only a report). The
  blocking-versus-advisory classification is knowledge and moves to the baseline.
- No standalone `implement` skill is created. The implementer already delegates
  to `tdd` and `committing`, so nothing reusable is left to extract, and PRD
  open-question #1 stays open.

### Consequences

- Good, because the baseline exists once and the composition graph stays readable
  from the manifests.
- Good, because ad hoc review no longer requires the `ship` loop, and the
  standalone and the cold PR reviewer cannot drift on their shared rubric.
- Good, because it resolves the follow-up ADR-0006 left open.
- Bad, because `ship:reviewer` now depends on an installed `code-review` plugin at
  runtime. A missing dependency disables the plugin, which is the loud failure
  ADR-0006 wants, but a failure nonetheless.
- Bad, because the shared knowledge and each reviewer's execution can still drift
  along the seam between them, the CI-green and verdict lines above.

## Pros and cons of the options

### review-baseline library skill (Packaging 1)

- Good, because knowledge lives once and execution stays per context.
- Good, because the reviewer fetches the baseline itself, honouring the
  cold-agent invariant rather than working around it.
- Bad, because it adds a third unit, a library skill, and a runtime dependency
  edge.

### The standalone skill is the baseline (Packaging 2)

- Good, because it is one fewer file.
- Bad, because loading it into the cold reviewer injects the in-session emission
  procedure, the conflicting-procedure smell the reviewer itself would flag.

### Invert control, `code-review` owns the reviewer agent

- Good, because the reviewer would exist exactly once, no duplication of any
  kind.
- Bad, because a workflow's agents ship inside its own plugin (`AGENTS.md`
  invariant), so `ship` briefing an agent it does not own reintroduces the
  skill-and-agent drift the bundle exists to prevent.
- Bad, because it rests on cross-plugin agent spawning, which is unverified here.

### Do nothing

- Bad, because it is the status quo, and the PRD and ADR-0006 both already name it
  a defect.
