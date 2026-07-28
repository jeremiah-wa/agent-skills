# 0016. Call skills by their namespaced name

- **Status:** Accepted
- **Date:** 2026-07-28

## Context and problem statement

[ADR-0006](0006-declare-invoked-skills-as-dependencies.md) made the skills
`ship:implementer` names **installed**. It did not make them the ones that
actually load.

A skill in `~/.claude/skills/` is itself loaded as a plugin (`claude plugin
init` scaffolds there and reports `<name>@skills-dir`), but it presents
**un-namespaced**. A plugin skill presents as `<plugin>:<skill>`. When both
exist, a bare call resolves to the loose one, silently. Measured on the
development machine, with `plugins/tdd` loaded:

| Call | `~/.claude/skills/tdd` present | Loaded |
|---|---|---|
| `Skill(tdd)` | yes | the third-party skill |
| `Skill(tdd)` | no | `plugins/tdd` |
| `Skill(tdd:tdd)` | yes | `plugins/tdd` |

So `ship:implementer` was reading a test-first discipline this repo did not
write, on the machine this repo is developed on, with `dependencies` correctly
declared and every manifest passing `claude plugin validate --strict`.

This is the same class of failure ADR-0006 was written to remove, in a shape it
did not anticipate: not a missing skill, but the wrong one. It also contradicts
the success criterion in [`PRD.md`](../PRD.md): "Nothing in this repo resolves by
accident of what happens to be installed."

Nothing warns. `claude doctor` reports no collision, and the two names appear
side by side in the skill listing without comment.

## Decision drivers

- The platform has **no collision detection**, so any guard has to be ours.
- [ADR-0004](0004-own-the-skills-we-use.md) earmarks `tdd`, `committing`,
  `code-review`, `implement`, `grilling` and more for this repo. Every one of
  those names already exists in `~/.claude/skills/`, so each is a live collision
  until switchover, not a hypothetical one.
- Collisions are **invisible at every gate**: the manifest is valid, the
  dependency resolves, the skill loads, and the agent reports success.
- [`GLOSSARY.md`](../GLOSSARY.md) already namespaces agents (`ship:implementer`)
  and every reference in the repo obeys it. Skills are the one component type
  still called bare.
- A human typing `/tdd` notices immediately when the wrong thing loads. A cold
  agent cannot notice, and its output looks equally confident either way.

## Considered options

- Namespace every call an agent makes, keep bare names for humans
- Rely on the one-name-one-home discipline alone
- Rename this repo's skills so no name can collide

## Decision outcome

Chosen option: "Namespace every call an agent makes, keep bare names for
humans", because it is the only option that is deterministic on a machine whose
contents we do not control, and it costs nothing at the human entry points.

Concretely: `Skill(tdd:tdd)` and `Skill(committing:committing)` in agent briefs
and skill bodies. `/tdd` and `/committing` remain the way a person invokes them.
The split is by audience, not by taste: a bare name is an affordance for someone
who can see what loaded.

Enforced mechanically in `.github/workflows/validate.yml`, since ADR-0006's
accepted cost ("prose and manifest can drift") is the same drift and is
checkable.

### Consequences

- Good, because a call resolves to the same skill on every machine, which is the
  `PRD.md` success criterion restated as a rule.
- Good, because the rule is greppable, so it is enforceable in CI rather than
  remembered.
- Good, because it makes the overlap period safe, which removes the pressure to
  rush the [ADR-0007](0007-private-until-switchover.md) switchover.
- Bad, because `tdd:tdd` and `committing:committing` stutter. That is the price
  of one-skill-per-plugin ([ADR-0002](0002-one-plugin-per-unit-grouped-by-cohesion.md))
  and is not worth renaming a plugin to avoid.
- Bad, because a plugin's **name** becomes API. Renaming a plugin now breaks
  call sites inside other plugins, where before it only broke `dependencies`.

## Pros and cons of the options

### Namespace every call an agent makes

- Good, because it is immune to whatever else is installed.
- Good, because it is mechanically checkable.
- Bad, because it is more verbose, and duplicates the plugin name in the
  single-skill case.

### Rely on the one-name-one-home discipline alone

- Good, because it needs no change to any call site.
- Good, because it is the right end state anyway, and switchover reaches it.
- Bad, because it is a rule about a machine, not about this repo, so it cannot
  be enforced by anything here and does not survive a fresh install elsewhere.
- Bad, because it fails silently the one time someone forgets.

### Rename this repo's skills so no name can collide

- Good, because bare calls would then be unambiguous.
- Bad, because the good names are the point: ADR-0004 chose to own `tdd` and
  `committing`, and a prefixed `ship-tdd` is worse at the `/` prompt for a
  permanent gain of nothing over namespacing.
