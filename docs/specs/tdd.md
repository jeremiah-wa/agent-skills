# `tdd`, Specification

What this plugin must **do** and **contain**, specified at the level another
agent needs to re-author it from a blank page. Canonical for this plugin's
contract ([ADR-0015](../decisions/0015-specs-are-one-file-per-plugin-under-docs.md)).
The spec shape and the repo-wide rules are in
[`docs/specs/README.md`](README.md).

**Current state.** This describes what is on disk today.

## The plugin

Single-skill plugin. The test-first discipline `ship:implementer` invokes, and a
user entry point in its own right. Authored from a blank page
([ADR-0005](../decisions/0005-author-from-a-blank-page.md)).

Manifest per-plugin values: `name` `tdd`; `version` `0.1.0`; `keywords`
`["testing", "tdd", "discipline"]`. No `skills` field: a root `SKILL.md` with no
`skills/` subdirectory picks itself up.

## `/tdd` (`SKILL.md`)

- **Frontmatter**: `name: tdd`; `description` (one line). Model-invocable, so no
  `disable-model-invocation`.
- **Responsibility**: hold the red-green-refactor loop, and hold the line on what
  a test is worth. The distinguishing opinion is that an unobserved failure is not
  a test.
- **Contract**:
  - **The loop.** Red: one test for one absent behaviour, run it, and the failure
    must be **the expected one**. A failure from an import error, a typo, or a
    missing fixture proves nothing, so repair the test until it fails for the
    right reason. Green: the least code that passes, because speed to green is
    what proves the test is wired to the thing under test. Refactor: clean up with
    the test holding behaviour still, then re-run. One behaviour at a time.
  - **Seams.** Define a **seam** as a boundary where behaviour is observable from
    outside (a return, a rendered output, a written record, an emitted event, a
    response body). Test there, because behaviour at a seam is exactly what
    survives refactoring. Name the failure mode explicitly: tests bound to
    implementation shape (a private call, a field set, a mock's messages) break on
    every rewrite, catch nothing, and teach the reader to distrust the suite. An
    unobservable behaviour is a **design finding**, reported rather than worked
    around.
  - **No seam.** Configuration, migrations, plumbing, and renames sometimes have
    nowhere to attach. Say so plainly in the commit or PR body and move on. Never
    invent a test to fill the slot: a test that cannot fail is a false green, and
    costs a reader more than an acknowledged gap.
  - **The bar.** Every test states a **failure scenario**: specific inputs or
    state leading to a specific wrong outcome it would catch. No statable
    scenario means the test is decoration. Prefer the unconsidered inputs (empty,
    zero, negative, missing, enormous, out of order, duplicated) over the happy
    path.
  - **Bugs.** A fix begins with a test that reproduces the bug, run before the fix
    and observed failing. An irreproducible bug is not yet understood, and the fix
    is a guess.
- **Edges**: none. It names no other skill and spawns nothing.
