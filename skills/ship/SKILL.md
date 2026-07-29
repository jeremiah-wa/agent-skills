---
name: ship
description: "Drive one issue to a reviewed draft PR: a cold implementer builds it, an adversarial reviewer attacks it."
disable-model-invocation: true
---

Take one unit of work from an issue to a PR that has survived an adversarial review.

You are the **orchestrator**. You do not write product code and you do not review it. You resolve the work item, brief and spawn the two agents, dispose of what the reviewer finds, and decide whether the loop continues.

Both agents are **cold**: they share no conversation with you or with each other. GitHub is the handoff medium. Anything one needs from the other has to be on the issue, the PR, or the review.

## 0. Preflight

Halt with a clear message if any check fails. Do not degrade to a local-only mode.

- `gh auth status` succeeds
- a `github.com` remote exists (`git remote -v`)
- the base branch resolves (`gh repo view --json defaultBranchRef`)
- the labels `agent:stop` and `agent:aborted` exist, or create them: `gh label create agent:stop --description "Halt the ship loop at the next round boundary"` and `gh label create agent:aborted --description "Ship loop halted before approval"`

`agent:stop` must exist before the loop starts, because it is how the user halts a run they are no longer watching.

If the **only** missing piece is the remote, offer `gh repo create` and push. Ask before running it, every time.

## 1. Resolve the work item

`/ship` takes an issue number, a path to a spec, or free text.

- **Issue number**: `gh issue view <n>`. Done.
- **Spec path or free text**: draft an issue body from it, show the user the full body, and create it with `gh issue create` once they approve. Never create an issue without showing the body first.

Everything downstream keys off the issue number.

## 2. Repo contract

Follow `Skill(ship:repo-contract)`. You cannot brief a cold agent without a contract.

## 3. Implement

Spawn `ship:implementer` with the issue number, the contract pasted in full, and the base branch. It branches, builds, pushes, and opens a **draft** PR.

Record the PR number and the agent's id. You resume this same agent every round, so it keeps its own reasoning.

Record the branch head as well: `gh pr view <pr> --json headRefOid`. Step 7 compares against it, so re-record it at the end of every round.

## 4. CI gate

First find out whether there are checks at all: `gh pr checks <pr> --json name`. An empty array means the repo has no CI, and `gh pr checks` reports that by exiting non-zero, which is indistinguishable from a failure if you do not look.

**With checks**, wait for them: `gh pr checks <pr> --watch`.

- **Green**: go to step 5.
- **Red**: send the failing logs (`gh run view <id> --log-failed`) to the implementer via `SendMessage`, then re-enter step 4. This is a free cycle and does not consume a review round, because no judgement was involved.
- **Red twice in a row**: breaker. Go to step 8 and halt.

**Without checks**, gate on the contract's verify commands instead: `SendMessage` the implementer to run them and report the output. A failure is red, and behaves exactly as above. The contract already carries verify commands for a repo with no CI, so this gate exists; it just runs in the implementer's worktree rather than on a runner.

Carry which gate ran into step 5. The reviewer skips lint, format, and types only because something already enforced them, and a self-reported local run is weaker evidence than a runner.

The reviewer only ever sees code that builds, lints, types, and passes tests, so its findings are about things the gate cannot see.

## 5. Review

Spawn a **fresh** `ship:reviewer` every round. Never resume one: independence is re-earned each time, not inherited.

Give it the PR number, the issue number, the contract, which gate ran in step 4, and the dispositions from every previous round so settled disputes are not relitigated.

It posts its own `gh pr review`. Read it back with `gh pr view <pr> --json reviews,comments`.

## 6. Dispose of every finding

Each finding gets exactly one disposition, on the record. Nothing disappears silently.

| Disposition | When | Action |
| --- | --- | --- |
| **fix** | the finding stands | `SendMessage` the finding to the implementer |
| **dispute** | the finding is wrong | reply on the review thread with your grounds |
| **defer** | real, but outside this issue's scope | `gh issue create`, then link it in a reply on the thread |

Blocking findings (confirmed correctness, and any spec finding) cannot be deferred. Fix them or dispute them.

Once the implementer has pushed, re-enter step 4.

## 7. Round boundary

Print a summary: round number, findings per axis, the disposition of each, what changed. Then check, in order:

1. **Circuit breakers.** Any one of these halts the run:
   - a verify command that will not execute at all (missing tool, broken bootstrap)
   - the diff has grown past 600 changed lines or 20 files without the issue calling for that scale
   - the same finding recurs across two rounds with no progress
   - CI red twice in a row
   - a force-push on the branch, which shows up as last round's head no longer being reachable: `git fetch`, then `git merge-base --is-ancestor <recorded-head> <current-head>` exits non-zero
2. **Stop signal.** `gh pr view <pr> --json labels` contains `agent:stop`.
3. **Round cap.** This was round 3.

All clear means the next round starts immediately. Do not wait for the user.

## 8. Exit

**Approved**: `gh pr ready <pr>`, `gh pr edit <pr> --add-assignee @me`, and post a summary comment covering rounds taken, findings by disposition, and any follow-up issues opened. Stop there. The user merges.

**Halted** (breaker, stop label, or round cap): leave the branch and the PR exactly as they are. Add the `agent:aborted` label, prefix the PR title with `WIP:`, and comment with the round it stopped at and what is still open. Tell the user the branch name and PR number.
