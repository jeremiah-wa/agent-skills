# AGENTS.md

Operating manual for agents working in this repo. Thin by design: it states what
the Claude Code docs and the skill-writing standard do not, and defers rather
than restates.

## Orientation

This repo curates Claude Code workflows. Each is a **plugin directory** under
`plugins/`, bundling its own `skills/` and `agents/`, listed in
`.claude-plugin/marketplace.json`. The content is prompts: no build, no test
suite, no runtime. The only mechanical check is `claude plugin validate
--strict`.

Why things are the way they are lives in [`docs/decisions/`](docs/decisions/).
What is true now lives in [`docs/`](docs/). Read before changing.

## Commands

```bash
claude plugin validate . --strict                    # the marketplace manifest
claude plugin validate ./plugins/<plugin> --strict   # a plugin manifest
claude plugin init <name> --with skills agents       # scaffold a new plugin
/reload-plugins                                      # after any change under agents/
```

`SKILL.md` edits apply live in the current session. Changes under `agents/`,
`hooks/`, and manifests need `/reload-plugins` or a restart.

## Invariants

- **A workflow's agents ship inside its plugin.** Never install an agent into
  `~/.claude/agents/`. The bundle stops a skill and its agent drifting apart.
- **No relative paths out of a plugin.** Installing copies the plugin into a
  version-keyed cache, so `../shared/` resolves to nothing.
- **Reach other plugins by name, not by file.** A manifest `dependencies` entry
  grants reachability of **named components**, never access to files
  ([ADR-0003](docs/decisions/0003-share-between-plugins-with-manifest-dependencies.md)).
  Content that genuinely must be read as a file lives inside one plugin.
- **Every skill named in a body or a brief is declared in `dependencies`.**
  Naming a skill that is not installed fails silently
  ([ADR-0006](docs/decisions/0006-declare-invoked-skills-as-dependencies.md)).
- **Agents call skills namespaced: `Skill(tdd:tdd)`, never `Skill(tdd)`.** A
  loose skill in `~/.claude/skills/` wins every bare-name call, so a bare call
  can load something this repo never wrote while every manifest still validates
  ([ADR-0016](docs/decisions/0016-call-skills-by-their-namespaced-name.md)). Bare
  names stay at the human entry points, where the wrong skill is visible.
- **A cold agent gets everything in its brief.** It shares no context with its
  spawner. Anything it needs is pasted in or fetched by it.
- **Preflight and refuse, do not degrade.** A workflow that needs a tool checks
  for it and stops clearly, rather than taking a quieter fallback that rots.

## House rules

- **No em dashes** anywhere (skills, agents, docs, commits, PR bodies). Use
  commas, parentheses, or separate sentences.
- **Prompt the positive.** State the target behaviour rather than banning the
  bad one; keep a prohibition only as a hard guardrail, paired with what to do
  instead.
- **Read the skill-writing standard before editing a `SKILL.md`.** Its
  vocabulary (information hierarchy, progressive disclosure, leading words,
  no-ops, sediment) is the house standard, not restated here.
- **Every line earns its place.** These are prompts in a context window, so
  prune no-op lines on every edit.
- **Author from a blank page.** Prior art is read and closed, never copied
  ([ADR-0005](docs/decisions/0005-author-from-a-blank-page.md)). Established
  vocabulary carries over; expression does not.

## Docs

Each doc owns one question. Nothing is restated across two of them.

| Doc | Owns | Normative? |
|---|---|---|
| [`docs/PRD.md`](docs/PRD.md) | Mission, problem, what belongs here and what does not. | Yes, for scope. |
| [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) | Repo shape, install and runtime resolution, boundaries, distribution. | Yes, for structure. |
| [`docs/specs/README.md`](docs/specs/README.md) | What a component spec must contain, the rule that nothing ships unspecified, and the index. | Yes, for the **shape** of a contract. |
| [`docs/specs/<plugin>.md`](docs/specs/) | That plugin's own components, to re-authoring depth. | **Yes. Canonical for its own components.** |
| [`docs/GLOSSARY.md`](docs/GLOSSARY.md) | The vocabulary to author in. | Yes, for terms. |
| [`docs/PRACTICE.md`](docs/PRACTICE.md) | How I work, and which skill serves each step. | No. Illustrative. |
| [`docs/decisions/`](docs/decisions/) | Why things changed. Append-only. | Yes, and immutable. |

The bar is **per component**: specified to the level another agent needs to
re-author it from a blank page
([ADR-0012](docs/decisions/0012-specify-per-component-not-per-repo.md)), in that
plugin's file under `docs/specs/`
([ADR-0015](docs/decisions/0015-specs-are-one-file-per-plugin-under-docs.md)).
Mechanical files (manifests, CI, `.gitignore`) are their own source and are not
restated in `docs/`. A plugin directory holds only what it ships.

### When to update which

A change is not finished until its row here is satisfied. Same commit, not a
follow-up.

| When you... | Update |
|---|---|
| Add a plugin | `.claude-plugin/marketplace.json`, a new `docs/specs/<plugin>.md`, the index in `docs/specs/README.md`, the contents table in `ARCHITECTURE.md`, the catalog in `README.md` |
| Add a component to an existing plugin | That plugin's `docs/specs/<plugin>.md`, and the components column of the index |
| Change a component's behaviour | Its section in `docs/specs/<plugin>.md`, and bump the plugin `version` |
| Add or change a `Skill()` call or a skill named in prose | The owning plugin's `dependencies`, and the component's **Edges** in its spec |
| Move a skill between owned, borrowed, local, or manual | `docs/PRACTICE.md` |
| Introduce or redefine a term | `GLOSSARY.md` |
| Change repo shape, or how anything resolves at install or runtime | `ARCHITECTURE.md` |
| Reverse or materially alter a stated invariant or commitment | A new ADR, **plus** the canonical doc it changes, linked back both ways |

Pure additions and clarifications are doc edits, not ADRs. See
[`docs/decisions/README.md`](docs/decisions/README.md).

## Adding a plugin

A plugin is a directory under `plugins/` with `.claude-plugin/plugin.json`, plus
an entry in `.claude-plugin/marketplace.json` giving its `name`, relative
`source` (`./plugins/<name>`), and `description`. Nothing else registers it
anywhere.

A plugin directory is self-contained. There is no shared tree it can reach into:
a `..` in its `skills` array is rejected by the validator as a path traversal
attempt
([ADR-0013](docs/decisions/0013-keep-plugins-in-a-plugins-directory.md)).

The manifest's mechanical fields are identical across plugins:

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
  "keywords": ["<...>"]
}
```

`skills` is **additive**, unlike `commands`, `agents`, and `outputStyles` which
replace their defaults. `skills/` is always scanned, so listing it is redundant
on its own. Add `"skills": ["./", "./skills/"]` only when a plugin has a root
`SKILL.md` **and** a `skills/` subdirectory: the auto-single-skill rule fires only
when there is no `skills/` subdirectory. A single-skill plugin omits the field.

Group by cohesion
([ADR-0002](docs/decisions/0002-one-plugin-per-unit-grouped-by-cohesion.md)):
skills that share reference files or always change together ship as one plugin.
Everything else ships alone.

## Workflow

- Conventional Commits scoped to the plugin (`feat(ship): ...`), scope omitted
  for repo-level changes.
- Branches `<type>/<slug>`, or `<type>/<issue#>-<slug>` where an issue exists.
- Bump a plugin's `version` when its behaviour changes; marketplace consumers
  only see updates when it moves.
