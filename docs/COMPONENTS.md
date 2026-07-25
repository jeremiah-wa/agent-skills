# agent-skills, Component Specifications

What every plugin and every component in this repo must **do** and **contain**,
specified at the level another agent needs to re-author it from a blank page.
This is the generative companion to [`ARCHITECTURE.md`](ARCHITECTURE.md) (which
gives the shape) and [`GLOSSARY.md`](GLOSSARY.md) (which gives the vocabulary),
and exists to meet the reconstruction bar of
[ADR-0011](decisions/0011-docs-must-reconstruct-the-repo.md). The recreation
procedure that ties them together is [`BUILD.md`](BUILD.md).

This is a **current-state** doc: it describes the components on disk today. A
decided-but-unbuilt change is flagged inline as *Pending ADR-NNNN* and specified
in the ledger, never silently folded in here.

## How to read a spec

Each component lists:

- **Path and frontmatter**: where the file goes and its exact YAML header.
- **Responsibility**: the one job it owns.
- **Contract**: what its body must encode, step by step or field by field. This
  is the requirement, not the wording. Two bodies that satisfy the same contract
  are both correct ([ADR-0005](decisions/0005-author-from-a-blank-page.md):
  expression is re-authored, never copied).
- **Edges**: what it spawns, loads, or depends on.

Author every body under the house rules in [`BUILD.md`](BUILD.md#agentsmd) and the
skill-writing standard ([ADR-0009](decisions/0009-own-the-skill-writing-standard.md)).

---

## Plugin: `ship`

Drives one issue to a reviewed draft PR. One plugin, not several, because its two
skills share `reference/repo-contract.md` and both drive its two agents (the
cohesion rule, [ADR-0002](decisions/0002-one-plugin-per-unit-grouped-by-cohesion.md)).

**`ship/.claude-plugin/plugin.json`** follows the plugin manifest template in
[`BUILD.md`](BUILD.md), which fixes the mechanical fields (`$schema`, `author`,
`repository`, `license`). Per-plugin values:

- `name`: `ship`
- `version`: `0.1.0`, bumped when behaviour changes
- `description`: one line, the issue-to-reviewed-PR summary
- `keywords`: `["workflow", "code-review", "github", "subagents"]`
- `skills`: `["./", "./skills/"]`. The `"./"` is load-bearing: a plugin with a
  root `SKILL.md` **and** a `skills/` subdirectory needs it, because the
  auto-single-skill rule only fires when there is no `skills/` subdirectory.
- *Pending [ADR-0010](decisions/0010-extract-the-review-baseline-into-a-library-skill.md):*
  add `dependencies: ["tdd", "committing", "code-review"]`.

### `/ship` — `ship/SKILL.md`

- **Frontmatter**: `name: ship`; `description` (one line); `disable-model-invocation: true`.
- **Responsibility**: the **orchestrator**. Writes no product code and reviews
  none. Resolves the work item, briefs and spawns the two agents, disposes of
  findings, decides whether the loop continues. Both agents are cold and share no
  conversation with it or each other; GitHub is the only handoff medium.
- **Contract** (the loop, in order):
  0. **Preflight.** Halt with a clear message on any failure; never degrade to a
     local-only mode. Check: `gh auth status` succeeds; a `github.com` remote
     exists; the base branch resolves (`gh repo view --json defaultBranchRef`);
     the labels `agent:stop` and `agent:aborted` exist or get created.
     `agent:stop` must exist before the loop starts, since it is how the user
     halts an unwatched run. If the **only** missing piece is the remote, offer
     `gh repo create` and push, asking before running it every time.
  1. **Resolve the work item.** Input is an issue number, a spec path, or free
     text. Issue number: `gh issue view <n>`. Spec or free text: draft an issue
     body, show the user the full body, create it with `gh issue create` only on
     approval. Everything downstream keys off the issue number.
  2. **Repo contract.** Follow `reference/repo-contract.md`. A cold agent cannot
     be briefed without it.
  3. **Implement.** Spawn `ship:implementer` with the issue number, the contract
     pasted in full, and the base branch. Record the PR number and the agent id;
     the same implementer is resumed every round so it keeps its reasoning.
  4. **CI gate.** `gh pr checks <pr> --watch`. Green: go to 5. Red: send the
     failing logs (`gh run view <id> --log-failed`) to the implementer, re-enter
     4. This is a free cycle, no review round consumed. Red twice in a row: a
     breaker, go to 8.
  5. **Review.** Spawn a **fresh** `ship:reviewer` every round, never resume one:
     independence is re-earned, not inherited. Give it the PR number, the issue
     number, the contract, and every previous round's dispositions.
  6. **Dispose of every finding.** Each finding gets exactly one disposition on
     the record: **fix** (`SendMessage` to the implementer), **dispute** (reply
     on the thread with grounds), or **defer** (`gh issue create`, link it on the
     thread). Blocking findings (confirmed correctness, and any spec finding)
     cannot be deferred. Re-enter 4 once the implementer has pushed.
  7. **Round boundary.** Print a summary (round number, findings per axis, each
     disposition, what changed). Then check in order: circuit breakers (see
     GLOSSARY); the `agent:stop` label; the round cap (round 3). All clear starts
     the next round immediately, without waiting for the user.
  8. **Exit.** Approved: `gh pr ready`, assign `@me`, post a summary comment; the
     user merges. Halted (breaker, stop label, or cap): leave branch and PR as
     they are, add `agent:aborted`, prefix the PR title with `WIP:`, comment with
     the round it stopped at and what is open, tell the user the branch and PR.
- **Edges**: spawns `ship:implementer` (resumed) and `ship:reviewer` (fresh each
  round); reads `reference/repo-contract.md`.

### `ship:implementer` — `ship/agents/implementer.md`

- **Frontmatter**: `name: implementer`; `description`; `model: sonnet`;
  `isolation: worktree`; `tools: [Read, Edit, Write, Bash, Grep, Glob, Skill]`.
- **Responsibility**: build one issue end to end in its own worktree, test-first,
  and hand it to a reviewer it never speaks to. Everything the reviewer needs
  must land in the commits, the PR body, or the code.
- **Contract**:
  - **Ground**: read the issue (`gh issue view <n>`) and the contract; read the
    invariant sources the contract names **before** touching anything (breaking an
    invariant is a bug even when tests pass); run the contract's **bootstrap**
    (the worktree is a fresh checkout).
  - **Branch** from the contract's base, following its branch pattern.
  - **Build** test-first at real seams via `Skill(tdd)`: a behavioural change
    ships with a test that fails without it. Where no seam exists (config,
    migration, plumbing), say so in the PR body rather than assert nothing. Run
    the contract's **verify** commands throughout; everything green locally before
    pushing, because CI gates whether a review happens. Commit via
    `Skill(committing)`, following the contract's commit pattern.
  - **Hand over**: push, open a **draft** PR against the base using the repo's PR
    template, referencing the issue so it closes on merge. Fill the template as a
    handover, not a summary: carry what the reviewer cannot get from the diff, the
    contract, or the issue, and answer any "what were you unsure about" honestly.
    Report the PR number, the branch, and anything not done.
  - **Later rounds**: resumed with CI failures or review findings, still holding
    its own earlier reasoning. Per finding: fix it, or state plainly why it is
    wrong; never apply a fix it does not believe in. Push each round as new
    commits on the same branch; never force-push (it destroys review anchors and
    trips a breaker).
- **Edges**: `Skill(tdd)`, `Skill(committing)` (Pending ADR-0006/0010: declared
  in `ship`'s `dependencies`).

### `ship:reviewer` — `ship/agents/reviewer.md`

- **Frontmatter (current)**: `name: reviewer`; `description`; `model: opus`;
  `isolation: worktree`; `tools: [Read, Bash, Grep, Glob]`.
  - *Pending [ADR-0010](decisions/0010-extract-the-review-baseline-into-a-library-skill.md):*
    add `Skill` to `tools`; replace the inline baseline (the stance, the twelve
    smells, and the verify discipline below) with `Skill(review-baseline)`;
    keep the execution shell (ground via `gh pr`, and the posting rules). Two
    lines stay here as execution, not knowledge: "CI is green, so lint, format,
    and types are not yours to report" (true only because `ship` gates on CI),
    and the request-changes-or-approve verdict (a `gh pr review` act).
- **Responsibility**: adversarial review. Presume a defect is present, hunt
  across correctness, spec, and standards, verify every suspicion against the
  code, and post a GitHub review. Being cold is the point: no reasoning it never
  heard can talk it out of a doubt. The counterweight to a free hunt is proof: an
  invented finding costs a real fix round and teaches the implementer to discount
  it.
- **Contract**:
  - **Ground**: `gh pr view <pr>` and `gh pr diff <pr>`; `gh issue view <n>`; the
    invariant sources in the brief, read in full; any prior-round dispositions.
    Form an own read of the whole diff **before** reading the PR body's claims,
    then treat those claims as claims to verify. Run the contract's bootstrap.
  - **Phase 1, hunt**: list every suspicion, unfiltered and unranked.
    - *Correctness*: behaviour on inputs nobody considered (empty, null, zero,
      negative, unicode, enormous); boundaries and off-by-ones; error paths and
      the state left after partial failure; ordering and concurrency; resource
      lifetimes; above all, any stated invariant broken.
    - *Spec*: what the issue asked for that is missing or half-built; what is here
      that nobody asked for; what looks implemented but is subtly wrong.
    - *Standards*: the repo's own documented standards. Skip anything the repo's
      tooling already enforces (CI is green, so formatting, lint, and types are
      out). On top of that, this smell baseline, every item a labelled judgement
      call rather than a hard violation, always overridden by a documented repo
      standard: **Mysterious Name**, **Duplicated Code**, **Feature Envy**, **Data
      Clumps**, **Primitive Obsession**, **Repeated Switches**, **Shotgun
      Surgery**, **Divergent Change**, **Speculative Generality**, **Message
      Chains**, **Middle Man**, **Refused Bequest**. (Each carries its one-line
      tell and its fix; see the source for the wording, re-author freely.)
  - **Phase 2, verify**: try to kill each suspicion by reading the path, running
    the test, or writing a throwaway reproduction. A suspicion survives only with
    a **failure scenario**: specific inputs or state leading to a specific wrong
    outcome. Mark survivors **CONFIRMED** (traced or reproduced) or **PLAUSIBLE**
    (reasoning holds, could not execute). Kill everything else; most of phase 1
    should die here.
  - **Post**: `gh pr review <pr>` with inline comments. **Request changes** when a
    confirmed correctness finding or any spec finding survived. **Approve** only
    when nothing blocking survived; standards findings are advisory and never
    block alone. Give each finding its axis, verdict, failure scenario, and fix,
    worst first. Leave alone any earlier finding whose dispute was sound. If
    nothing was found, show the hunt: what was tried and how.
- **Knowledge/execution split** (the basis of Pending ADR-0010): the stance, the
  twelve smells, the verify-and-failure-scenario discipline, and the
  blocking-versus-advisory classification are **knowledge** (they move to
  `review-baseline`). Grounding via `gh pr`, the CI-green skip, and the
  `gh pr review` verdict are **execution** (they stay here).

### `/grill-pr` — `ship/skills/grill-pr/SKILL.md`

- **Frontmatter**: `name: grill-pr`; `description`; `disable-model-invocation: true`.
- **Responsibility**: point `ship`'s adversarial reviewer at a PR that already
  exists, with no implement loop. For PRs opened by hand, by someone else, or by
  another tool.
- **Contract**:
  1. Resolve the PR: `gh pr view <n> --json number,headRefName,baseRefName,body,url`;
     with no number, use the current branch's PR.
  2. Find the spec: the issue the PR closes, or a path the user passed. If none,
     say so in the brief and the reviewer skips the spec axis rather than invent
     one.
  3. Get the repo contract (follow `../../reference/repo-contract.md`).
  4. Spawn `ship:reviewer` with the PR number, the spec, and the contract pasted
     in full.
  5. Report what it posted, then stop. Fixing is not this skill's job, and neither
     is arguing with the findings; the author decides.
- **Edges**: spawns `ship:reviewer`; reads `reference/repo-contract.md`. Cannot
  move out of `ship`: it spawns a `ship` agent, so relocating it into a plugin
  `ship` depends on would cycle.

### `repo-contract` — `ship/reference/repo-contract.md`

- **Responsibility**: define the **contract**, the facts a cold agent needs to
  work in a repo without guessing. It lives in the target repo's `AGENTS.md`
  under a `## Agent contract` heading, so it is version controlled and corrected
  once rather than re-guessed every run.
- **Contract**:
  - *If it exists*: read it and paste it verbatim into every agent brief.
  - *If it does not*: probe each field, then confirm with the user before use.
    Fields and where to probe: **base** (`gh repo view --json defaultBranchRef`);
    **bootstrap** (the install step in `.github/workflows/*.yml` first, then the
    lockfile plus manifest); **verify** (the commands CI runs, authoritative;
    fall back to a manifest script block only with no CI); **branch**
    (`AGENTS.md`, `CONTRIBUTING.md`, or infer from `git branch -a`); **commit**
    (`AGENTS.md`, `CONTRIBUTING.md`, `.gitmessage`, or infer from `git log`);
    **invariants** (`AGENTS.md`, `CLAUDE.md`, `docs/ARCHITECTURE.md`,
    `docs/decisions/`); **pr template** (`.github/PULL_REQUEST_TEMPLATE.md`).
    Show the user the whole proposed contract, ask for corrections, write it into
    `AGENTS.md` on approval (creating a thin `AGENTS.md` if the repo has none).
  - Never run on an unconfirmed contract: every round inherits its errors, and a
    wrong verify command costs a full cycle before anything notices.
  - Include a worked **Shape** example of the `## Agent contract` block.

---

## Planned components (decided, not yet on disk)

Specified in full in
[ADR-0010](decisions/0010-extract-the-review-baseline-into-a-library-skill.md)
and the [`PRD`](PRD.md) first pass. Summarised here so the map is complete; do
not build these when reproducing the current repo, only when applying the
pending decision.

- **Plugin `tdd`** — single-skill plugin, the test-first discipline
  `ship:implementer` invokes. Authored from a blank page, the third-party `tdd`
  as prior art only.
- **Plugin `committing`** — single-skill plugin, the atomic-commit and
  Conventional-Commit discipline `ship:implementer` invokes. Same authoring rule.
- **Plugin `code-review`** — two skills sharing the review domain:
  - `skills/review-baseline` — the review **knowledge** (stance, twelve smells,
    verify-and-failure-scenario discipline, blocking-versus-advisory
    classification), `disable-model-invocation: true`. Loaded by name, a library.
  - `SKILL.md` (`/code-review`) — an in-session wrapper that loads
    `review-baseline` and reviews the working-tree diff by default, falling back
    to the merge-base with the default branch, accepting an explicit base, and
    skipping the spec axis when no spec is given (as `grill-pr` does).
