# 0001. Distribute as Claude Code plugins through a marketplace

- **Status:** Accepted
- **Date:** 2026-07-25

## Context and problem statement

This collection has to install on any machine I work from. Two candidate
channels exist. Claude Code's native plugin marketplace (`/plugin marketplace
add owner/repo`), and the `skills` CLI
([vercel-labs/skills](https://github.com/vercel-labs/skills),
`npx skills@latest add owner/repo`), which is already how third-party skills
reach this machine. The channel has to be chosen before the repo layout can be,
because each channel expects a different one.

## Decision drivers

- `ship`, the only workflow here today, is built from two subagents
  (`ship:implementer`, `ship:reviewer`). Without them it does nothing.
- The `skills` CLI distributes **skills only**. Agents do not travel with them.
- The `skills` CLI expects a flat `skills/<name>/SKILL.md` layout and installs
  per-skill folders, so content shared between two skills by relative path is
  lost.
- Working from harnesses other than Claude Code (Cursor, Codex, Amp) is not a
  requirement.
- Install must not depend on machine-specific setup.

## Considered options

- Claude Code marketplace only
- `npx skills` only
- Multi-harness via `npx skills`, redesigning `ship` to not need subagents
- Both channels from the same repo

## Decision outcome

Chosen option: "Claude Code marketplace only", because it is the only channel
that carries agents, and every workflow here is built out of agents.

### Consequences

- Good, because agents, hooks, manifests, versioning, and per-plugin
  enable/disable are all available.
- Good, because it unlocks the `dependencies` field as the sharing mechanism
  (see [ADR-0002](0002-one-plugin-per-unit-grouped-by-cohesion.md) and
  [ADR-0003](0003-share-between-plugins-with-manifest-dependencies.md)).
- Good, because "installable on any machine" reduces to pushing the repo, not to
  adopting new tooling.
- Bad, because these skills are Claude Code only. Reaching another harness would
  need a second channel and a second layout.
- Bad, because auto-update is off by default for non-Anthropic marketplaces, so
  a machine picks up changes via `claude plugin update` or by opting into
  auto-update in `/plugin`.

## Revisit when

The decision is to stay on Claude Code, the only harness in use here, and
revisit if that changes. This note records why so the question is not
re-litigated.

Codex CLI has since grown native subagents (`.codex/agents/*.toml`, worktree
isolation, per-agent roles), so the implementer/reviewer pattern is no longer
Claude Code only. But it is not a harness in use here, and the outcome stands:
the `skills` CLI still distributes `SKILL.md` only, carrying no agent definition
to any harness, and the load-bearing agent fields (`model`, the enforced `tools`
whitelist, `isolation`) do not port. The `model` pin in particular is the whole
model-selection mechanism, and it lives in the agent, not the skill.

The hope is that this repo grows to support multiple harnesses if and when a
second harness comes into use here, and a mechanism to distribute agents across
harnesses (a skills-CLI that carries agents, or a shared agent-spec a generator
can compile) exists. Until both are true, the Claude-Code-native files already
in this repo are the canonical source such a mechanism would consume, so keeping
the door open costs nothing today.

## Pros and cons of the options

### Claude Code marketplace only

- Good, because agents ship with the skills that spawn them.
- Good, because manifest `dependencies` are enforced at install and load time.
- Bad, because it forecloses other harnesses.

### `npx skills` only

- Good, because the install one-liner is shorter and needs no marketplace step.
- Bad, because agents do not travel, so `ship` would install looking healthy and
  fail at its first spawn.
- Bad, because per-skill install breaks relative references between sibling
  skills, such as `grill-pr` reading `../../reference/repo-contract.md`.

### Multi-harness via `npx skills`

- Good, because the same skills would work from Cursor and Codex.
- Bad, because `ship` would have to be rebuilt without subagents, which removes
  the cold-agent independence that is the point of it.

### Both channels

- Good, because each workflow could pick the channel that suits it.
- Bad, because two layouts must be satisfied at once, and a skill that works in
  one channel can break silently in the other.
