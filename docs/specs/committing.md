# `committing`, Specification

What this plugin must **do** and **contain**, specified at the level another
agent needs to re-author it from a blank page. Canonical for this plugin's
contract ([ADR-0015](../decisions/0015-specs-are-one-file-per-plugin-under-docs.md)).
The spec shape and the repo-wide rules are in
[`docs/specs/README.md`](README.md).

**Current state.** This describes what is on disk today.

## The plugin

Single-skill plugin. The atomic-commit and Conventional-Commit discipline
`ship:implementer` invokes, and a user entry point in its own right. Authored
from a blank page, and the first skill migrated in from a machine-local
directory.

Manifest per-plugin values: `name` `committing`; `version` `0.1.0`; `keywords`
`["git", "commits"]`. No `skills` field.

## `/committing` (`SKILL.md`)

- **Frontmatter**: `name: committing`; `description` (one line). Model-invocable.
- **Responsibility**: one commit does one thing, and its message carries the why.
  The audience is a reader of `git log` a year later, and a reviewer reading
  commits in order before the diff.
- **Contract**:
  - **Before committing**: read `git status` and `git diff --staged`. Never stage
    with `git add -A` on trust. Catch two things: files that wandered in (logs,
    editor artifacts, `.env`), and two changes sharing one commit.
  - **Splitting**: an "and" in the message means two commits. A refactor and the
    behaviour change it enabled are separate, because a no-behaviour refactor is
    reviewable at a glance and unreviewable when tangled. A test and the code it
    covers stay **together**. Renames, formatting passes, and dependency bumps go
    alone. Stage by path where the split is clean, `git add -p` where it is not.
  - **The message**: `<type>(<scope>): <subject>` with a blank line before the
    body. Types `feat`, `fix`, `docs`, `refactor`, `test`, `chore`, `build`, `ci`,
    `perf`. Scope in the target repo's own vocabulary, omitted when the change is
    repo-wide. Subject imperative, lower case, no trailing period, under about 70
    characters.
  - **The body**: says **why**, the only part unreconstructable from the diff.
    Write one when a reason is worth keeping (the constraint that forced the
    shape, the approach abandoned, the thing that looks wrong and is not). Omit it
    when the subject is the whole story; a body restating the subject is noise.
    Reference an issue where one exists.
  - **House rules**: no em dashes; no `Co-Authored-By` or other co-author or tool
    attribution; never `--no-verify`, since a failing hook is a finding; prefer a
    new commit to amending, and amend only what is unpushed.
  - **On behalf of someone**: commit only what was asked for, leave unrelated
    work as found and say so, never commit or push unbidden, and branch first when
    on the default branch.
- **Edges**: none.
