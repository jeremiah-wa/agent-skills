# 0017. Restructure around a flat `skills/` tree

- **Status:** Proposed
- **Date:** 2026-07-29

## Context and problem statement

[ADR-0013](0013-keep-plugins-in-a-plugins-directory.md) recorded that a shared
`skills/` tree "does **not** resolve", on the evidence that
`claude plugin validate` rejects a `..` segment in a plugin's `skills` array.
That evidence is real but it tested the wrong shape. It tested `..` escaping
**out of** `plugins/<name>/`. When a marketplace entry sets `source: "./"`, the
plugin root **is** the marketplace root, so `./skills/<name>` needs no `..` and
validates clean. The option ADR-0013 closed is open.

Two further facts arrived with it. The [Agent Skills
specification](https://agentskills.io/specification) defines no plugin, no
marketplace, and no manifest: a skill is a directory holding `SKILL.md` plus
optional `scripts/`, `references/`, and `assets/`, and `name` must match the
parent directory. Everything under `.claude-plugin/` is a Claude Code
distribution layer sitting on top of a deliberately packaging-agnostic format.
And the spec requires a skill's file references to resolve **from the skill
root**, which `plugins/ship/skills/grill-pr/SKILL.md:15` violates: it reaches
`../../reference/repo-contract.md`, outside its own root.

So the layout question is live again, and it now has a portability argument
behind it that it did not have in ADR-0013.

## Decision drivers

- **The skill is the portable unit; the plugin is the vendor wrapper.** A layout
  that keeps spec-shaped skill directories survives a consumer that is not
  Claude Code. `plugins/<name>/SKILL.md` at the plugin root conforms only
  because each plugin happens to be named after its one skill.
- **`grill-pr` is not currently a valid standalone skill.** Any option has to
  resolve `repo-contract.md`, and that resolution differs per option.
- **Two safety nets currently come for free.** Skill frontmatter is validated
  when the validator is pointed at a plugin directory, and each plugin's version
  is isolated from the others. Both are load-bearing and both are at risk.
- **`marketplace.json` entries are more capable than the repo assumes.**
  Confirmed by trial: `strict: false` lets an entry define a plugin whole, and
  `skills`, `agents`, and `dependencies` all work from an entry. Installing an
  entry resolves and installs its declared dependencies.
- **Enable and disable remain per-plugin.** Every skill in an enabled plugin
  pays metadata cost in every session, roughly 100 tokens
  ([ADR-0002](0002-one-plugin-per-unit-grouped-by-cohesion.md) measured 80 to
  100). At five skills that is about 500 tokens, which is now a concrete number
  rather than an unbounded fear.
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

Chosen option: **pending**. This record exists to hold the comparison and the
trial evidence; the choice between B and C is the open question it is written to
settle.

The recommendation is **C**, on the grounds that it delivers the flat
spec-shaped tree, which is the portability goal, while keeping both safety nets
intact, and that the cost it charges (about 500 tokens per session) is smaller
than the cost B charges (rebuilding frontmatter validation and version isolation
by hand, and never being allowed to let either lapse).

**This flips if per-plugin enable/disable is a requirement rather than a
preference.** B is the only option that keeps it alongside the flat tree. If a
user must be able to take `tdd` without `ship`, C is disqualified and the
correct answer is B with all four of its mitigations funded up front.

### Consequences

Common to B and C:

- Good, because every skill becomes a spec-conformant directory whose `name`
  matches its parent structurally rather than by coincidence.
- Good, because `grill-pr` stops reaching outside its own root, so it is
  portable for the first time.
- Bad, because the authoring junctions on every machine must be repointed, and
  a junction to `skills/<name>` carries `SKILL.md` but not the repo-root
  `agents/`, so the authoring loop for `ship` needs rechecking.
- Bad, because ADR-0013 and ADR-0002 both quote `plugins/`-era paths. Records
  are append-only, so those stand as written and are read as historical.
- Neutral: [ADR-0016](0016-call-skills-by-their-namespaced-name.md) is
  unaffected. The namespace is the plugin name, which under B comes from the
  marketplace entry and under C is the single plugin.

Specific to B:

- Bad, because `claude plugin validate . --strict` no longer validates skill
  frontmatter at all. Confirmed by trial: a `SKILL.md` with no `description`,
  an uppercase name, consecutive hyphens, and a name not matching its directory
  passed clean, exit 0. The same file fails validation today when the validator
  is pointed at a plugin directory. CI would go green on a skill that cannot
  load. `skills-ref validate` becomes mandatory, not optional.
- Bad, because every plugin shares `source: "./"` and therefore shares the repo
  HEAD as its version. Confirmed by trial: a commit touching only one skill
  bumped an untouched plugin from `e65f83adf898` to `9a61dfbd9152`. Avoiding
  this requires an explicit `version` on every entry, bumped by hand, which is
  the manual discipline the platform docs warn produces stale versions that mask
  real changes.
- Bad, because `marketplace.json` becomes the single point of truth and failure
  for every plugin's wiring. What a plugin ships with stops being answerable
  from its directory.
- Bad, because `validate.yml` needs a genuine rewrite rather than a path edit:
  the plugin loop at lines 29 to 44 iterates nothing, and the `Skill()` checker
  at lines 53 to 54 reads `plugins/$plugin/.claude-plugin/plugin.json`, which
  will not exist. The guarantee in
  [ADR-0006](0006-declare-invoked-skills-as-dependencies.md) is only as good as
  that script.
- Bad, because migration is all-or-nothing. `strict: false` alongside a
  `plugin.json` that declares components is a load failure, not a merge.

Specific to C:

- Good, because `repo-contract.md` stays a file at the repo root, shared freely,
  with no mechanism and no behavioural change.
- Good, because one `plugin.json` means frontmatter validation keeps working
  unchanged and there is one version to bump.
- Bad, because enable and disable become all-or-nothing for the whole
  collection, which is the tax ADR-0002 was written to avoid.
- Bad, because the internal dependency graph collapses. ADR-0003 and ADR-0006
  become moot, and the `Skill()` CI check has nothing left to verify beyond
  namespacing.
- Bad, because a future consumer who wants only `committing` has no way to take
  only `committing`.

If either is accepted, ADR-0002 and ADR-0013 need `Superseded by ADR-0017`, and
`ARCHITECTURE.md` needs its Shape, Resolution, and Boundaries sections rewritten.

## Pros and cons of the options

### B. Flat tree, marketplace entries are the authority

Verified working end to end: validates `--strict`, installs, and installing an
entry resolved and installed its two declared dependencies.

- Good, because filepath stops determining packaging, which was the original
  goal. Adding a skill is one directory and one line.
- Good, because one skill can appear in several entries, which the current
  layout cannot express at all.
- Good, because it keeps per-plugin enable and disable, the property ADR-0002
  correctly identified as the platform's real currency.
- Good, because it is what `anthropics/skills` does in production: 16 skills in
  a flat tree, zero `plugin.json` files, three entries slicing it.
- Bad, because it silently loses frontmatter validation (see Consequences).
- Bad, because it couples every plugin's version to the repo HEAD.
- Bad, because `source: "./"` copies the entire repo into the cache once per
  entry. Confirmed by trial: three entries produced three complete copies,
  including skills each entry does not load. Noise at text-file scale; not noise
  if `scripts/` ever carries dependencies.
- Bad, because `--plugin-dir`, local-path installs, and bare clones stop working
  permanently. ADR-0001 already chose marketplace-only, so this costs nothing
  today, but it closes a door rather than leaving it ajar.
- Bad, because it makes [ADR-0010](0010-extract-the-review-baseline-into-a-library-skill.md)
  a prerequisite rather than a plan: `repo-contract` becomes a skill, and
  `ship:reviewer` has no `Skill` tool with which to reach it.
- Bad, because a shared file becoming a skill is a behavioural change. A link is
  read deterministically; a skill is invoked by model decision.

### C. One plugin, whole repo

The shape `vercel/vercel-plugin` uses: repo root is the plugin, `source: "./"`,
41 skills in one flat `skills/`, one `plugin.json`, `strict` left alone.

- Good, because it gets the flat spec-shaped tree while keeping a real
  `plugin.json`, so standalone installability survives.
- Good, because one cache copy, one version, one manifest, and the simplest
  possible CI.
- Good, because it needs no mitigations funded before it is safe to adopt.
- Bad, because it costs per-plugin enable and disable outright.
- Bad, because every skill pays metadata cost in every session whether or not
  the user wants it.
- Bad, because this shape only exists at N=1. A shared `skills/` tree with
  several `plugin.json` files is impossible: each manifest would need
  `../skills/<name>`, which is the `..` ADR-0013 correctly found is rejected.

### A. Keep `plugins/<name>/`, fix only the violations

- Good, because nothing breaks, no record is superseded, and it is half a day.
- Good, because its fixes (resolve the `../../` escape, rename `reference/` to
  `references/`, add `license` and `compatibility` to frontmatter, add
  `skills-ref validate` to CI) are required under B, C, and D alike, so it is
  the correct first step regardless of what follows.
- Bad, because it delivers none of what prompted the question: the flat tree,
  reuse of a skill across plugins, and filepath-independence.

### D. Split into two repos

A vendor-neutral skills repo with no `.claude-plugin/` at all, plus a thin
marketplace repo whose entries use a `github` source.

- Good, because the skills become consumable by anything implementing the spec,
  with Claude Code as one consumer among several.
- Bad, because it buys portability for a need that does not exist. ADR-0007
  keeps the collection private, so there are no other consumers to serve.
- Bad, because two repos means two release cadences and cross-repo version
  pinning.
