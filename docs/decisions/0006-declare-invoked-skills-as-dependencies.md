# 0006. Declare the skills an agent invokes as plugin dependencies

- **Status:** Accepted
- **Date:** 2026-07-25

## Context and problem statement

`ship/agents/implementer.md` calls `Skill(tdd)` and `Skill(committing)`. Neither
skill ships in this repo, and neither is declared anywhere. They resolved on the
development machine only by accident of what was installed there:

- `tdd` came from `mattpocock-skills`.
- `committing` was a 38-line `SKILL.md` sitting in `~/.claude/skills/committing/`,
  not a symlink, not in any marketplace, not under version control anywhere. It
  existed on exactly one machine.

So `ship` was installable on one computer, and neither install channel would have
reported a problem. The failure would have surfaced as an agent quietly not
doing the thing.

## Decision drivers

- Cross-plugin composition happens by **naming a skill in prose**, which is how
  the platform intends it and works across plugin boundaries at runtime.
- Naming a skill that is not installed fails **silently**. There is no load-time
  check on prose.
- A manifest `dependencies` entry is the only declaration Claude Code enforces:
  a missing one disables the plugin with `dependency-unsatisfied` and prints the
  install command.
- `AGENTS.md` already states "Preflight and refuse, do not degrade" as an
  invariant. An undeclared skill dependency is precisely a silent degradation.
- A subagent whose `tools` list omits `Skill` cannot reach a skill at all.
  `ship:reviewer` is such an agent (`Read, Bash, Grep, Glob`).

## Considered options

- Declare every named skill in the owning plugin's `dependencies`
- Inline the guidance into the agent brief and drop the `Skill()` calls
- Leave them undeclared and rely on the machine having them

## Decision outcome

Chosen option: "Declare every named skill in the owning plugin's
`dependencies`", because it converts a silent runtime gap into a loud install-time
error, which is the behaviour the repo already claims to want.

Concretely: `ship` declares `dependencies: ["tdd", "committing"]` once those
skills exist here as single-skill plugins per
[ADR-0002](0002-one-plugin-per-unit-grouped-by-cohesion.md).

### Consequences

- Good, because installing `ship` on a clean machine either works or says exactly
  what is missing.
- Good, because it makes the composition graph readable from the manifests rather
  than only from prose.
- Bad, because an agent without the `Skill` tool still needs its content inline in
  the agent file or pasted into its spawn brief. `ship:reviewer` carries its
  review baseline this way, which is why the same material currently appears in
  two places (unresolved, see `AGENTS.md` follow-ups).
- Bad, because prose and manifest can drift. A named skill added to a brief
  without a matching manifest entry reintroduces the original bug.

## Pros and cons of the options

### Declare in `dependencies`

- Good, because the platform enforces it at install and at load.
- Good, because `--prune` cleans up what is no longer required.
- Bad, because it must be maintained alongside the prose that names the skill.

### Inline the guidance

- Good, because the plugin becomes fully standalone with no dependencies at all.
- Bad, because it duplicates content that already exists as a skill, and the
  agent brief grows.

### Leave undeclared

- Bad, because it is the status quo, and the status quo is the bug.
