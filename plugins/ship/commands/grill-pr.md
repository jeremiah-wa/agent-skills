---
description: "Point the adversarial reviewer at an existing PR: hunt, verify, post a review, stop."
---

Run `ship`'s adversarial reviewer against a pull request that already exists, without the implement loop.

Use it on PRs opened by hand, by someone else, or by another tool.

## Process

1. **Resolve the PR.** `gh pr view <n> --json number,headRefName,baseRefName,body,url`. With no number given, use the PR for the current branch.
2. **Find the spec.** The issue the PR closes, or a path the user passed. If there is none, say so in the brief and the reviewer skips the spec axis rather than inventing one.
3. **Get the repo contract.** Follow [`../skills/ship/references/repo-contract.md`](../skills/ship/references/repo-contract.md).
4. **Spawn `ship:reviewer`** with the PR number, the spec, and the contract pasted in full.
5. **Report what it posted**, then stop. Fixing is not your job here, and neither is arguing with the findings: the author decides.
