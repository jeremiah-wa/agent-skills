# 0002. One plugin per unit, grouped by cohesion

- **Status:** Accepted
- **Date:** 2026-07-25

## Context and problem statement

Given the marketplace channel ([ADR-0001](0001-distribute-as-claude-code-plugins.md)),
the unit of install is a plugin. The repo's title promises workflows **and**
skills, but a standalone skill has nowhere to live unless the layout says where.
How many skills go in a plugin?

## Decision drivers

- Enable and disable are **per-plugin**, never per-skill. A plugin is taken whole
  or not at all.
- Every skill in an enabled plugin pays an **always-on context cost** in every
  session, whether it fires or not. `claude plugin list` prints it per component;
  the documented worked example is roughly 80 to 100 tokens per skill.
- The `dependencies` field can only name a **plugin**. A skill inside a bag
  cannot be depended on individually.
- Skills inside one plugin can share files by relative path. Across plugins they
  cannot (see
  [ADR-0003](0003-share-between-plugins-with-manifest-dependencies.md)).
- A plugin's `version` covers everything in it, so any change bumps the whole
  thing.

## Considered options

- Cohesion rule: a plugin holds skills that share files or always change together
- Always one plugin per skill
- One bag per theme (`github`, `writing`, `review`)
- Decide per case, no rule

## Decision outcome

Chosen option: "Cohesion rule", because the real axis is not how many skills a
plugin holds but whether those skills share files and change together, and that
is the only property the platform actually rewards.

Concretely: a plugin holds skills that share reference files or always change
together. Anything independently useful gets its own single-skill plugin, with
`SKILL.md` at the plugin root so the frontmatter `name` sets a clean invocation
name.

`ship` therefore stays whole. It and `grill-pr` share
`ship/reference/repo-contract.md`, and both depend on the two agents in
`ship/agents/`.

### Consequences

- Good, because `ship` keeps its shared reference file with no mechanism needed.
- Good, because independently useful skills stay independently installable and
  cost context only when enabled.
- Good, because single-skill plugins are the only unit `dependencies` can name,
  which is what makes ADR-0003 workable.
- Bad, because every addition is a judgement call rather than a mechanical rule,
  so the rule has to be restated where it can be found.
- Bad, because N plugins means N manifests, N version numbers, and N marketplace
  entries.

## Pros and cons of the options

### Cohesion rule

- Good, because it matches what the platform charges for: context per skill,
  dependencies per plugin, file sharing per plugin.
- Bad, because it needs judgement, and judgement drifts.

### Always one plugin per skill

- Good, because it is uniform and maximally composable.
- Bad, because it would force `grill-pr` out of `ship` and duplicate
  `repo-contract.md`, or make `grill-pr` depend on all of `ship` anyway to reach
  `ship:reviewer`.

### One bag per theme

- Good, because fewer plugins to manage than one per skill.
- Bad, because the context tax scales with the theme's size regardless of what
  is used, and nothing inside a theme can be depended on individually.

### No rule

- Good, because maximum flexibility per addition.
- Bad, because the layout drifts and the same argument is had every time.
