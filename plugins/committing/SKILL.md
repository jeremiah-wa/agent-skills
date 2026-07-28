---
name: committing
description: "Craft atomic git commits with Conventional Commit messages. Use when creating a commit, splitting work into commits, or writing a commit message."
---

One commit does **one thing**, and its message says why.

The audience is someone reading `git log` in a year with no memory of this work, and a reviewer reading the commits in order to understand the change before they read the diff.

## Before you commit

Read what you are about to commit: `git status` and `git diff --staged`. Never stage with `git add -A` and trust it.

Two things to catch: files that wandered in (a stray log, an editor artifact, a `.env`), and **two changes sharing one commit**.

## Splitting

If the message needs an "and", it is two commits.

Split along the seams that already exist in the work:

- A refactor and the behaviour change it enabled are separate. The refactor is reviewable at a glance when it changes no behaviour, and impossible to review when it is tangled with something that does.
- A test and the code it covers belong **together**. That pair is one thing.
- A rename, a formatting pass, or a dependency bump goes alone. Mixed into a real change, it buries the part worth reading.

Stage by path where the split is clean. Where it is not, `git add -p`.

## The message

Conventional Commits:

```
<type>(<scope>): <subject>

<body>
```

- **type**: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`, `build`, `ci`, `perf`.
- **scope**: the part of the repo affected, in that repo's own vocabulary. Omit it for changes that are genuinely repo-wide.
- **subject**: imperative, lower case, no trailing period. "add the preflight check", not "added" or "adds".

Keep the subject under about 70 characters. It is read in a list.

## The body

The subject says what. The body says **why**, and it is the only part a reader cannot reconstruct from the diff.

Write one when there is a reason worth keeping: the constraint that forced this shape, the approach that was tried and abandoned, the thing that looks wrong and is not. Skip it entirely when the subject is the whole story. A body restating the subject is noise.

Reference an issue when one exists (`refs #12`, or `closes #12` where merging should close it).

## House rules

- **No em dashes.** Use commas, parentheses, or separate sentences.
- **No `Co-Authored-By` trailers**, and no other co-author or tool attribution. The commit is the author's.
- **Never `--no-verify`.** A failing hook is a finding. Fix the cause.
- **Prefer a new commit to amending.** Amend only what you have not pushed.

## Committing on behalf of someone

Commit only what was asked for. If you notice unrelated work sitting in the tree, say so and leave it staged or unstaged as you found it, rather than sweeping it into the commit.

Never commit or push unless it was asked for. If the current branch is the default branch, branch first.
