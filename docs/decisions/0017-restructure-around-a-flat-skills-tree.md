# 0017. Restructure around a flat `skills/` tree

- **Status:** Rejected
- **Date:** 2026-07-29

## Context and problem statement

[ADR-0013](0013-keep-plugins-in-a-plugins-directory.md) recorded that a shared
`skills/` tree "does **not** resolve", on the evidence that
`claude plugin validate` rejects a `..` segment in a plugin's `skills` array.
That evidence is real but it tested the wrong shape. It tested `..` escaping
**out of** `plugins/<name>/`. When a marketplace entry sets `source: "./"`, the
plugin root **is** the marketplace root, so `./skills/<name>` needs no `..` and
validates clean. The option ADR-0013 closed looked open again.

Two further facts arrived with it. The [Agent Skills
specification](https://agentskills.io/specification) defines no plugin, no
marketplace, and no manifest: a skill is a directory holding `SKILL.md` plus
optional `scripts/`, `references/`, and `assets/`, and `name` must match the
parent directory. Everything under `.claude-plugin/` is a Claude Code
distribution layer sitting on top of a deliberately packaging-agnostic format.
And the spec requires a skill's file references to resolve **from the skill
root**, which `plugins/ship/skills/grill-pr/SKILL.md:15` violated: it reached
`../../reference/repo-contract.md`, outside its own root.

So the layout question was reopened with a portability argument behind it that
it did not have in ADR-0013. Every option below was then built, installed, and
measured. The trials are what closed it.

## Decision drivers

These are the drivers as they stood when the question was reopened. Two of them
did not survive the trials, and are corrected in place below.

- **The skill is the portable unit; the plugin is the vendor wrapper.** A layout
  that keeps spec-shaped skill directories survives a consumer that is not
  Claude Code. `plugins/<name>/SKILL.md` at the plugin root conforms only
  because each plugin happens to be named after its one skill.
- **`grill-pr` is not currently a valid standalone skill.** Any option has to
  resolve `repo-contract.md`, and that resolution differs per option.
- **Two safety nets currently come for free.** Skill frontmatter is validated
  when the validator is pointed at a plugin directory, and each plugin's version
  is isolated from the others. Both are load-bearing and both are at risk.
- **~~`marketplace.json` entries are more capable than the repo assumes.~~
  Corrected.** `strict: false` does let an entry define a plugin whole, and
  `skills` and `dependencies` do work from an entry. **`agents` does not, and
  neither does `commands`.** Both validate clean from an entry and load nothing.
  See the measurement under "B" below. This driver was the load-bearing one for
  option B and it was wrong.
- **Enable and disable remain per-plugin.** Every skill in an enabled plugin
  pays metadata cost in every session, roughly 100 tokens
  ([ADR-0002](0002-one-plugin-per-unit-grouped-by-cohesion.md) measured 80 to
  100). This turned out to be the wrong unit to count in: agents cost more than
  skills do, and under B they are not per-plugin at all.
- **The collection is still private** ([ADR-0007](0007-private-until-switchover.md)),
  so relative `source` paths are not yet cached by anyone and moving directories
  is still cheap. It gets monotonically more expensive.

## Considered options

- **B.** Flat `skills/` tree, marketplace entries are the authority
  (`strict: false`, no `plugin.json` anywhere)
- **C.** One plugin, whole repo (flat `skills/` tree, a single `plugin.json` at
  the repo root, `strict` left at its default)
- **A.** Keep `plugins/<name>/` and fix only the spec violations
- **D.** Split into a vendor-neutral skills repo and a thin marketplace repo

## Decision outcome

Chosen option: **A, keep `plugins/<name>/`**, because the restructure's two
premises did not survive being built. B costs roughly twice what C costs while
being the option that loses the safety nets, and the spec violation that
prompted the question turned out to be independent of the layout entirely.

The proposal in this record's title is therefore **rejected**. Nothing moves.
`plugins/<name>/` stays, one `plugin.json` per plugin stays, and
[ADR-0002](0002-one-plugin-per-unit-grouped-by-cohesion.md) and
[ADR-0013](0013-keep-plugins-in-a-plugins-directory.md) both stand as written
and remain `Accepted`. Neither is superseded.

Option A's fixes proceed on their own, tracked as issues rather than folded in
here, because none of them needs a layout change to land.

### The trials

Every shape below was built as a branch, installed from a local marketplace, and
measured with `claude plugin details`. The branches are named so the evidence is
reproducible.

| Branch | Shape | All three installed | `committing` alone |
|---|---|---|---|
| `trial/0017-option-b-original` | B as this record specified it | ~686 tok | ~192 tok |
| `trial/0017-nested-skills` | B with `grill-pr` nested under `ship` | ~655 tok | ~216 tok |
| `trial/0017-one-plugin` | C | ~339 tok | not possible |
| `trial/0017-status-quo-plus-command` | `plugins/`, `grill-pr` as a command | ~351 tok | ~56 tok |
| `trial/0017-skill-inside-plugin` | `plugins/<name>/skills/<name>/` | ~351 tok | ~56 tok |

Figures are the always-on cost added to every session, summed across the three
plugins where they install separately.

Four findings came out of this, in descending order of how much they changed the
answer.

**`agents` and `commands` on a marketplace entry are inert.** They validate
clean, load nothing, and suppress nothing. An entry declaring
`"agents": ["./skills/ship/agents/reviewer.md"]` reported `Agents (0)`; the same
files at the plugin root reported `Agents (2)`. Declaring `"agents": []`
suppressed nothing either. Under `source: "./"` the plugin root **is** the repo
root, so every agent and command in the repo loads into every entry with no
opt-out. Measured: `tdd` came back `Agents (2) implementer, reviewer` and ~202
tokens for a skill that is ~66 tokens on its own.

That is what inverts the cost comparison. This record recommended C on the
grounds that its roughly 500 tokens per session was the price of keeping the
safety nets. C measures ~339 and B measures ~686, and B is the option without
the safety nets.

**Per-plugin enable and disable is the property B was chosen for, and B only
half delivers it.** B isolates skills and globalises agents. Keeping
`plugins/<name>/` isolates both, at ~351 rather than ~686.

**Nesting a skill inside another skill's directory buys nothing.** An entry
declaring `./skills/ship` reported `Skills (1)`; the nested
`./skills/ship/skills/grill-pr` had to be listed explicitly to load, which is
exactly what a flat tree would have cost. Directory nesting is presentational.

**`skills-ref validate` checks frontmatter and nothing else.** A skill whose body
reaches `../outside/contract.md` passes clean. The spec violation this record
opens with is real, and no available tool enforces it. What the tool does catch
on this repo is `disable-model-invocation`, a Claude Code extension the spec's
frontmatter allow-list rejects.

A fifth finding is unrelated to the layout and is tracked separately: the
marketplace `name` field, `agent-skills`, is on Claude Code's reserved list, so
`claude plugin marketplace add` has never worked against this repo.

### Consequences

- Good, because nothing moves. No path reference, no junction on any machine, no
  `source` in `marketplace.json`, and no record needs superseding.
- Good, because both safety nets are kept without having to fund a replacement.
  Frontmatter is validated by pointing the validator at a plugin directory, and
  each plugin's version stays isolated from the others.
- Good, because the question is now settled on measurements rather than on
  reasoning about the platform, and the branches keep the measurements
  reproducible.
- Bad, because the portability goal is unmet. `plugins/<name>/` conforms to the
  spec because each plugin is named after its one skill, which is a coincidence
  the layout does not enforce. Making that structural means
  `plugins/<name>/skills/<name>/`, which measures identically and was not
  adopted here only because withdrawing `grill-pr` removed the reason it
  mattered.
- Bad, because a skill still cannot appear in more than one plugin. Nothing in
  the repo needs that today, and ADR-0007 means there are no other consumers to
  need it either, so the cost is deferred rather than paid.
- Neutral: [ADR-0016](0016-call-skills-by-their-namespaced-name.md) is
  unaffected. The namespace is the plugin name, which is unchanged.

## Pros and cons of the options

### A. Keep `plugins/<name>/`, fix only the violations

Measured on `trial/0017-status-quo-plus-command`: ~351 tokens for all three,
~56 for `committing` alone, `Agents (0)` on the plugins that ship none.

- Good, because it is the only option that isolates both skills **and** agents
  per plugin, which is what the platform actually charges for.
- Good, because nothing breaks, no record is superseded, and the authoring
  junctions on every machine keep resolving.
- Good, because its fixes are required under B, C, and D alike, so it was always
  the correct first step regardless of what followed.
- Bad, because it delivers none of what prompted the question: the flat tree,
  reuse of a skill across plugins, and filepath-independence.
- Bad, because spec conformance rests on the naming coincidence described above
  rather than on structure.

### B. Flat tree, marketplace entries are the authority

Validates `--strict`, installs, and installing an entry resolves and installs
its declared dependencies. Everything below is measured, not predicted.

- Good, because filepath stops determining packaging, which was the original
  goal. Adding a skill is one directory and one line.
- Good, because one skill can appear in several entries, which the current
  layout cannot express at all.
- Good, because it is what `anthropics/skills` does in production: 16 skills in
  a flat tree, zero `plugin.json` files, three entries slicing it. That repo
  ships no agents, so it never meets the finding below.
- Bad, because **agents and commands are repo-global and cannot be scoped**.
  This is the finding that decided it. `committing` alone costs ~192 tokens for
  a ~56-token skill, because `ship`'s two agents are in it whether or not
  anyone wants them.
- Bad, because it is the **most expensive** of the shapes measured, ~686 against
  ~351, since the agents load once per entry.
- Bad, because it keeps per-plugin enable and disable only for skills, which
  undercuts the one property that made it worth considering.
- Bad, because `claude plugin validate . --strict` no longer validates skill
  frontmatter at all. A `SKILL.md` with no `description`, an uppercase name,
  consecutive hyphens, and a name not matching its directory passed clean, exit
  0. CI would go green on a skill that cannot load.
- Bad, because every plugin shares `source: "./"` and therefore shares the repo
  HEAD as its version. A commit touching one skill bumped an untouched plugin
  from `e65f83adf898` to `9a61dfbd9152`. Avoiding this needs an explicit
  `version` on every entry, bumped by hand.
- Bad, because `source: "./"` copies the entire repo into the cache once per
  entry. Three entries produced three complete copies, each carrying `docs/`,
  `.github/`, and the other entries' skills.
- Bad, because it forces `repo-contract` to become a skill, which is a
  behavioural change and not only a move. A link is read deterministically every
  time; a skill is invoked when the model decides to. `ship`'s preflight is
  built on "you cannot brief a cold agent without a contract".
- Bad, because `marketplace.json` becomes the single point of truth and failure
  for every plugin's wiring, and `validate.yml` needs a rewrite rather than a
  path edit.
- Bad, because `--plugin-dir`, local-path installs, and bare clones stop working
  permanently.

### C. One plugin, whole repo

The shape `vercel/vercel-plugin` uses: repo root is the plugin, `source: "./"`,
41 skills in one flat `skills/`, one `plugin.json`, `strict` left alone.
Measured at ~339 tokens on `trial/0017-one-plugin`.

- Good, because it gets the flat spec-shaped tree while keeping a real
  `plugin.json`, so standalone installability survives.
- Good, because one cache copy, one version, one manifest, and the simplest
  possible CI.
- Good, because it is the cheapest shape measured, though only by ~12 tokens
  against keeping `plugins/`.
- Bad, because it costs per-plugin install outright, and ~12 tokens is not a
  price worth taking that for.
- Bad, because the internal dependency graph collapses. ADR-0003 and ADR-0006
  become moot, and the `Skill()` CI check has nothing left to verify beyond
  namespacing.
- Bad, because this shape only exists at N=1. A shared `skills/` tree with
  several `plugin.json` files is impossible: each manifest would need
  `../skills/<name>`, which is the `..` ADR-0013 correctly found is rejected.

### D. Split into two repos

A vendor-neutral skills repo with no `.claude-plugin/` at all, plus a thin
marketplace repo whose entries use a `github` source.

- Good, because the skills become consumable by anything implementing the spec,
  with Claude Code as one consumer among several.
- Bad, because it buys portability for a need that does not exist. ADR-0007
  keeps the collection private, so there are no other consumers to serve.
- Bad, because two repos means two release cadences and cross-repo version
  pinning.

## Corrections to this record

Two claims made when this record was written did not survive the trials. They
are struck through above rather than deleted, because the record is the account
of a decision and the wrong turns are part of it.

- **`agents` works from a marketplace entry.** It does not. Neither does
  `commands`. Both validate and load nothing. This was the driver B rested on.
- **`license` and `compatibility` must be added to skill frontmatter.** They are
  optional. `plugins/tdd` and `plugins/committing` pass `skills-ref validate`
  without either. What actually fails is `disable-model-invocation`.
