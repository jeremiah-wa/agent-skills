# agent-skills, Architecture

How this repo is put together and how its contents resolve at install and at
runtime. Terms are defined in [`GLOSSARY.md`](GLOSSARY.md). Reasons live in
[`decisions/`](decisions/). What each component must do is specified in
[`specs/`](specs/), one file per plugin, and how I work is mapped in
[`PRACTICE.md`](PRACTICE.md).

## Shape

The repo **is** a Claude Code marketplace. Its root holds repo-level concerns
only; every plugin is one directory under `plugins/`
([ADR-0013](decisions/0013-keep-plugins-in-a-plugins-directory.md)).

```
agent-skills/
├── .claude-plugin/
│   └── marketplace.json              # lists every plugin, by relative source path
├── .github/workflows/validate.yml
├── AGENTS.md                         # contributor contract, thin, defers
├── README.md
├── docs/                             # this doc set
└── plugins/
    ├── committing/                   # a single-skill plugin
    │   ├── .claude-plugin/plugin.json
    │   └── SKILL.md
    ├── tdd/                          # a single-skill plugin
    │   ├── .claude-plugin/plugin.json
    │   └── SKILL.md
    └── ship/                         # a plugin with several components
        ├── .claude-plugin/plugin.json
        ├── SKILL.md                  # the plugin's own root skill: /ship
        ├── skills/grill-pr/SKILL.md  # a second skill in the same plugin
        ├── agents/                   # ship:implementer, ship:reviewer
        └── reference/repo-contract.md
```

Adding a plugin means adding a directory under `plugins/` and an entry in
`marketplace.json`. Nothing else is registered anywhere.

A plugin directory is **self-contained**. There is no shared `skills/` tree that
plugins draw from: a `..` in a plugin's `skills` array is rejected by
`claude plugin validate` as a path traversal attempt, before install ever enters
into it.

## Current contents

| Plugin | Entry points | Components |
|---|---|---|
| `ship` | `/ship`, `/grill-pr` | 2 skills, 2 agents, 1 shared reference file |
| `tdd` | `/tdd` | 1 skill |
| `committing` | `/committing` | 1 skill |

`ship` is one plugin rather than two because `SKILL.md` and `skills/grill-pr/`
both read `reference/repo-contract.md`, and both drive the same two agents. That
is the cohesion rule from
[ADR-0002](decisions/0002-one-plugin-per-unit-grouped-by-cohesion.md).

## Manifest rules that matter

- `skills` **adds to** the default. `skills/` is always scanned, so listing it
  explicitly is redundant.
- A plugin with `SKILL.md` at its root **and** a `skills/` subdirectory needs
  `"skills": ["./"]` to pick up the root one. The auto-single-skill rule applies
  only when there is no `skills/` subdirectory. `ship` is in the first case, so
  its `"./"` entry is load-bearing.
- `commands`, `agents`, and `outputStyles` **replace** their defaults when
  specified. `skills` does not.

## Resolution

Three different places hold skills on a development machine, and they behave
differently.

| Location | How it gets there | Behaviour |
|---|---|---|
| Plugin cache | `claude plugin install` from a marketplace | Version-keyed copy. Paths leaving the plugin root do not resolve. Symlinks resolving elsewhere in the same marketplace are dereferenced and copied in; symlinks leaving the marketplace are skipped. |
| `~/.claude/skills/<name>` | A junction created while authoring, or the `skills` CLI | Loaded in place as `<name>@skills-dir`. One entry per name, so a junction placed here replaces whatever held that name. |
| `~/.agents/skills/<name>` | The `skills` CLI (`npx skills`), tracked in `~/.agents/.skill-lock.json` | Symlinked into `~/.claude/skills/`. Third-party only; nothing in this repo installs this way. |

`SKILL.md` edits apply live in the current session. Changes to `agents/`,
`hooks/`, and manifests do not, and need `/reload-plugins` or a restart.

## Boundaries

Four constraints shape everything above.

1. **No relative paths out of a plugin root.** `claude plugin validate` rejects
   a `..` segment in a plugin's `skills` array outright, and installing copies
   the plugin into a cache where external files were never placed anyway.
2. **A dependency does not share files.** It installs another plugin into its
   own cache directory. Sharing reaches **named components**, not file content.
   Real file sharing requires the sharers to be in one plugin.
3. **Agents ship inside the plugin that spawns them.** Never installed into
   `~/.claude/agents/`. The bundle is what stops a skill and its agent drifting
   apart.
4. **An agent reaches a skill only if its `tools` list includes `Skill`.**
   `ship:implementer` has it; `ship:reviewer` does not, so anything the reviewer
   needs is inline in its agent file or pasted into its brief.
   ([ADR-0010](decisions/0010-extract-the-review-baseline-into-a-library-skill.md)
   decides to give the reviewer `Skill` and move its baseline to a
   `review-baseline` library skill; pending implementation.)

## Composition

Skills reach other skills by **naming them in prose**. This works across plugin
boundaries because it resolves at runtime through the `Skill` tool, with no path
resolution involved.

The failure mode is silence: a named skill that is not installed produces
nothing. The manifest `dependencies` array is the only declaration the platform
enforces, so every skill named in a skill body or an agent brief is declared
there. See [ADR-0006](decisions/0006-declare-invoked-skills-as-dependencies.md).

Today, with `code-review` still to be built
([ADR-0010](decisions/0010-extract-the-review-baseline-into-a-library-skill.md)
adds it, and the `Skill` tool to `ship:reviewer` so it can load
`review-baseline`):

```
ship ──depends──> tdd
     ├─depends──> committing
     └─depends──> code-review   (planned; reviewer loads review-baseline)
```

## Distribution

Marketplace only
([ADR-0001](decisions/0001-distribute-as-claude-code-plugins.md)).

```
/plugin marketplace add jeremiah-wa/agent-skills
/plugin install ship@agent-skills
```

Auto-update is off by default for non-Anthropic marketplaces. A machine picks up
changes with `claude plugin update <plugin>` then `/reload-plugins`, or by
enabling auto-update for the marketplace in `/plugin`.

Authoring uses a junction instead, so edits apply without reinstalling:

```powershell
cmd /c mklink /J "$HOME\.claude\skills\ship" "D:\Wajer\repos\agent-skills\plugins\ship"
```

## Validation

There is no build, no test suite, and no runtime. The content is prompts. The
only mechanical check is manifest validation, run in CI on push and pull
request:

```bash
claude plugin validate . --strict                   # the marketplace manifest
claude plugin validate ./plugins/<plugin> --strict  # each plugin manifest
```

This is worth stating plainly because it bounds what this repo can test about
its own workflows: `ship` gates a review round on CI and its implementer works
test-first, and neither has much to bite on here. See
[ADR-0008](decisions/0008-document-with-current-state-docs-and-a-ledger.md).

Beyond the mechanical check there is one non-mechanical test: a step of the
practice that works only on the machine it was written on is not yet owned
([ADR-0012](decisions/0012-specify-per-component-not-per-repo.md)).
[`PRACTICE.md`](PRACTICE.md) maps every step of every workflow to the skill
serving it and marks whether that skill is in this repo. It runs by reading, not
in CI.
