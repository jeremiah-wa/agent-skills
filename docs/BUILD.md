# Reconstructing agent-skills from `docs/`

`docs/` is meant to be the complete generative source for this repo
([ADR-0011](decisions/0011-docs-must-reconstruct-the-repo.md)). Delete everything
else and an agent reading only `docs/` can rebuild it. This file is the entry
point: the procedure, the completeness checklist, and the root scaffolding that
lives nowhere else.

## What "recreated" means

Not byte-identical. [ADR-0005](decisions/0005-author-from-a-blank-page.md) makes
prose re-authored from a blank page, so two `SKILL.md` bodies that satisfy the
same contract in [`COMPONENTS.md`](COMPONENTS.md) are both correct. The bar is
**behavioural and structural equivalence**:

- Every file in the manifest below exists.
- `claude plugin validate . --strict` and each plugin validate clean.
- Every plugin, entry point, agent, and component behaves as its spec requires.
- Every decision in [`decisions/`](decisions/) still holds.

**Mechanical files are the exception**: manifests, the CI workflow, and
`.gitignore` are not prose, so reproduce them verbatim from the specs here.

## Procedure

1. Read [`decisions/`](decisions/) end to end. The invariants and the reasons
   live there, and they constrain everything else.
2. Read [`ARCHITECTURE.md`](ARCHITECTURE.md) for the shape and the resolution
   rules, and [`GLOSSARY.md`](GLOSSARY.md) for the vocabulary to author in.
3. Read [`COMPONENTS.md`](COMPONENTS.md) for what each plugin and component does.
4. Create the root scaffolding (below).
5. Create each plugin from its spec in `COMPONENTS.md`: manifest first, then
   skills and agents.
6. Author every body under the house rules (below) and the skill-writing standard
   ([ADR-0009](decisions/0009-own-the-skill-writing-standard.md)).
7. Validate with the commands under [Commands](#commands). Fix until clean.

To rebuild the **current** repo, stop there. To rebuild the **decided** repo,
then apply the pending changes flagged in `COMPONENTS.md` and specified in
[ADR-0010](decisions/0010-extract-the-review-baseline-into-a-library-skill.md).

## File manifest

Everything outside `docs/`. This is the "did I miss anything" checklist.

```
.gitignore
README.md
AGENTS.md
.claude-plugin/marketplace.json
.github/workflows/validate.yml
ship/.claude-plugin/plugin.json
ship/SKILL.md
ship/agents/implementer.md
ship/agents/reviewer.md
ship/skills/grill-pr/SKILL.md
ship/reference/repo-contract.md
```

Component specs for everything under `ship/` are in
[`COMPONENTS.md`](COMPONENTS.md). The five root files are specified below.

## Root scaffolding

### `.gitignore` (verbatim)

```
.DS_Store
Thumbs.db
*.log
.claude/settings.local.json
```

### `.github/workflows/validate.yml` (verbatim)

```yaml
name: Validate

on:
  push:
    branches: [main]
  pull_request:

concurrency:
  group: validate-${{ github.ref }}
  cancel-in-progress: true

jobs:
  validate:
    name: Manifests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5

      - uses: actions/setup-node@v4
        with:
          node-version: "22"

      - name: Install Claude Code
        run: npm install -g @anthropic-ai/claude-code

      - name: Validate marketplace
        run: claude plugin validate . --strict

      - name: Validate plugins
        run: |
          for plugin in */; do
            if [ -f "$plugin/.claude-plugin/plugin.json" ]; then
              echo "::group::$plugin"
              claude plugin validate "$plugin" --strict
              echo "::endgroup::"
            fi
          done
```

### `.claude-plugin/marketplace.json`

The repo **is** the marketplace. Mechanical, so reproduce it verbatim; the only
part that changes over time is the `plugins` array, one `{ name, source,
description }` entry per plugin directory (`source` is the relative path). Today
that is `ship` alone.

```json
{
  "$schema": "https://anthropic.com/claude-code/marketplace.schema.json",
  "name": "agent-skills",
  "description": "Curated Claude Code workflows. Each plugin bundles its own skills and agents so a workflow ships as one piece.",
  "owner": {
    "name": "Jeremiah Wangaruro",
    "email": "jeremiahwa98@gmail.com"
  },
  "plugins": [
    {
      "name": "ship",
      "source": "./ship",
      "description": "Issue to reviewed draft PR: a cold implementer builds it, an adversarial reviewer attacks it, GitHub carries the handoff"
    }
  ]
}
```

### Plugin manifests, `<plugin>/.claude-plugin/plugin.json`

Every plugin carries one. The fields below are mechanical and identical across
plugins, so reproduce them verbatim; the per-plugin fields (`name`, `version`,
`description`, `keywords`, `skills`) are filled from the plugin's spec in
[`COMPONENTS.md`](COMPONENTS.md). `skills` is `["./", "./skills/"]` only when a
plugin has both a root `SKILL.md` and a `skills/` subdirectory; a single-skill
plugin (root `SKILL.md` only) omits `skills` entirely.

```json
{
  "$schema": "https://anthropic.com/claude-code/plugin.schema.json",
  "name": "<plugin>",
  "version": "0.1.0",
  "description": "<one line>",
  "author": {
    "name": "Jeremiah Wangaruro",
    "email": "jeremiahwa98@gmail.com"
  },
  "repository": "https://github.com/jeremiah-wa/agent-skills",
  "license": "MIT",
  "keywords": ["<...>"],
  "skills": ["./", "./skills/"]
}
```

### `README.md`

Public-facing. The catalog is grouped by plugin, the install unit
([ADR-0002](decisions/0002-one-plugin-per-unit-grouped-by-cohesion.md)), split
into `## Workflows` (plugins that coordinate agents) and `## Skills`
(single-purpose plugins) as a reading aid; a library skill is noted as internal,
not an entry point. Show only the sections that have plugins, and treat the rows
in the template as examples: today that is `## Workflows` with `ship` alone, and
`## Skills` is omitted until a single-purpose plugin exists. `PRD.md` becomes this
file once it settles
([ADR-0008](decisions/0008-document-with-current-state-docs-and-a-ledger.md)),
carrying the mission with it. The template:

````markdown
# agent-skills

A curated collection of Claude Code workflows, packaged as plugins.

More than a toolbox: this repo is the documented specification of how its owner
engineers, and the skills are how that practice compiles today. See
[`docs/PRD.md`](docs/PRD.md#mission) for the mission.

Each workflow is a self-contained plugin directory bundling its own skills **and**
its own agents, so the pieces ship and version together and cannot drift apart.

## Workflows

Plugins that coordinate agents.

| Plugin | Entry points | What it does |
| --- | --- | --- |
| [`ship`](ship/) | `/ship`, `/grill-pr` | <one line> |

## Skills

Single-purpose plugins. A library skill (`disable-model-invocation`) is internal,
not an entry point.

| Plugin | Entry points | What it does |
| --- | --- | --- |
| [`<name>`](<name>/) | `/<name>` | <one line> |

## Install

**For authoring** (loads in place, `SKILL.md` edits apply live):

```bash
ln -s /path/to/agent-skills/<plugin> ~/.claude/skills/<plugin>
```

On Windows use a junction (no elevation, no Developer Mode). Git Bash's `ln -s`
silently copies instead, which looks identical and never picks up an edit:

```powershell
cmd /c mklink /J "$HOME\.claude\skills\<plugin>" "D:\path\to\agent-skills\<plugin>"
```

**For use elsewhere**, via the marketplace:

```
/plugin marketplace add jeremiah-wa/agent-skills
/plugin install <plugin>@agent-skills
```

## Requirements

`ship` needs the `gh` CLI authenticated against a GitHub remote. It preflights and
refuses rather than degrading to a local-only mode.

## Validate

```bash
claude plugin validate . --strict
claude plugin validate ./<plugin> --strict
```
````

### `AGENTS.md`

The canonical contributor contract, single-sourced here so it survives the
deletion test. Reconstruct the root `AGENTS.md` as the **thin pointer** form: a
short file with orientation, the command block, and a one-line index of the rules
below, all deferring here for the detail. That is the live form. The full template
below is the canonical content the pointer defers to, not a second copy to place
at the root.

<a id="commands"></a>

````markdown
# AGENTS.md

Operating manual for agents working in this repo. Thin by design: it states what
the Claude Code docs and the skill-writing standard do not, and defers rather than
restates.

## Orientation

This repo curates Claude Code workflows. Each is a **plugin directory** at the
repo root bundling its own `skills/` and `agents/`, listed in
`.claude-plugin/marketplace.json`. The content is prompts: no build, no test
suite, no runtime. The only mechanical check is `claude plugin validate --strict`.

## Commands

```bash
claude plugin validate . --strict               # the marketplace manifest
claude plugin validate ./<plugin> --strict      # a plugin manifest
claude plugin init <name> --with skills agents  # scaffold a new plugin
/reload-plugins                                 # after any change under agents/
```

`SKILL.md` edits apply live in the current session. Changes under `agents/`,
`hooks/`, and manifests need `/reload-plugins` or a restart.

## Invariants

- **A workflow's agents ship inside its plugin.** Never install an agent into
  `~/.claude/agents/`. The bundle stops a skill and its agent drifting apart.
- **No relative paths out of a plugin.** Installing copies the plugin into a
  cache, so `../shared/` resolves to nothing. Shared content is duplicated or
  symlinked, never referenced upward.
- **A cold agent gets everything in its brief.** It shares no context with its
  spawner. Anything it needs is pasted in or fetched by it.
- **Preflight and refuse, do not degrade.** A workflow that needs a tool checks
  for it and stops clearly, rather than taking a quieter fallback that rots.

## House rules

- **No em dashes** anywhere (skills, agents, docs, commits, PR bodies). Use
  commas, parentheses, or separate sentences.
- **Prompt the positive.** State the target behaviour rather than banning the bad
  one; keep a prohibition only as a hard guardrail, paired with what to do
  instead.
- **Read the skill-writing standard before editing a `SKILL.md`.** Its vocabulary
  (information hierarchy, progressive disclosure, leading words, no-ops, sediment)
  is the house standard, not restated here.
- **Every line earns its place.** These are prompts in a context window, so prune
  no-op lines on every edit.

## Workflow

- Conventional Commits scoped to the plugin (`feat(ship): ...`), scope omitted for
  repo-level changes.
- Branches `<type>/<slug>`, or `<type>/<issue#>-<slug>` where an issue exists.
- Bump a plugin's `version` when its behaviour changes; marketplace consumers only
  see updates when it moves.
````

## The `docs/` source set

What survives the deletion and generates the rest:

| Doc | Role |
|---|---|
| [`BUILD.md`](BUILD.md) | This file: recreation procedure, manifest, root scaffolding. |
| [`ARCHITECTURE.md`](ARCHITECTURE.md) | Repo shape, resolution, boundaries, distribution. |
| [`COMPONENTS.md`](COMPONENTS.md) | Per-plugin, per-component behavioural specs. |
| [`GLOSSARY.md`](GLOSSARY.md) | The vocabulary to author in. |
| [`PRD.md`](PRD.md) | Problem, solution, scope. Temporary; becomes `README.md`. |
| [`decisions/`](decisions/) | The append-only ledger of why. |
