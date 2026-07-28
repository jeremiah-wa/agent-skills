# agent-skills, Glossary

The vocabulary this repo is designed in. Use these terms consistently in skills,
agent briefs, docs, and commits.

Ambiguity here has already cost real design time: one proposed component name
covered two opposite roles and the conflict stayed invisible until dependency
direction was questioned. See
[ADR-0008](decisions/0008-document-with-current-state-docs-and-a-ledger.md).

## Packaging

| Term | Meaning |
|---|---|
| **Plugin** | The unit of install, versioning, and enable/disable. A directory with `.claude-plugin/plugin.json`, holding skills, agents, and reference files. Nothing smaller can be installed, enabled, disabled, or depended on. |
| **Marketplace** | A repo with `.claude-plugin/marketplace.json` listing plugins. This repo is one, named `agent-skills`. |
| **Plugin cache** | Where an installed plugin's files actually live, keyed by version. Paths traversing outside a plugin root do not resolve here, which is why `../shared/` never works after install. |
| **Single-skill plugin** | A plugin whose whole content is one `SKILL.md` at its root. The frontmatter `name` sets the invocation name, so it reads as `/name`. The only unit `dependencies` can name precisely. |
| **Cohesion rule** | The layout rule from [ADR-0002](decisions/0002-one-plugin-per-unit-grouped-by-cohesion.md): a plugin holds skills that share reference files or always change together. Everything else ships alone. |
| **Dependency** | A `dependencies` entry in `plugin.json` naming another plugin. Bare names resolve within the same marketplace with no tags and no config. **Grants reachability of named components, never access to files.** |
| **Bundle plugin** | A plugin that is only a `dependencies` array, existing to install a curated set in one command. Depends **on** everything. Not yet decided for this repo. |
| **Library** | The opposite arrow: a plugin that everything else depends **on**. One plugin cannot be both a bundle and a library, that is a cycle. |
| **Always-on context cost** | Tokens every enabled skill's description spends in every session, whether it fires or not. Roughly 80 to 100 tokens per skill. The brake on putting many skills in one plugin. |

## Authoring and install

| Term | Meaning |
|---|---|
| **Junction** | A Windows directory link (`mklink /J`), needing no elevation and no Developer Mode. How this repo's plugins are loaded in place while being authored, so `SKILL.md` edits apply live. Git Bash's `ln -s` silently copies instead, which looks identical and never picks up an edit. |
| **Skills-dir plugin** | A plugin loaded from `~/.claude/skills/<name>` rather than from a marketplace, appearing as `<name>@skills-dir`. The authoring mode here. |
| **Switchover** | The moment `mattpocock-skills` is uninstalled and this collection stands alone. No fixed trigger; my judgement. Also the trigger for making this repo public ([ADR-0007](decisions/0007-private-until-switchover.md)). |
| **Prior art** | A third-party skill read for reference and then closed, never copied, whether met at the bootstrap or adopted later once earned by use. Ideas and vocabulary carry over; expression does not ([ADR-0005](decisions/0005-author-from-a-blank-page.md)). |
| **Owned / borrowed / local / manual** | The four statuses a step of the practice can have in [`PRACTICE.md`](PRACTICE.md). **Owned**: the skill is a plugin here. **Borrowed**: a third-party skill installed on the machine. **Local**: authored by me but living in `~/.claude/skills/` on one machine only, unversioned. **Manual**: no skill runs the step. Only **owned** is portable; the other three all mean the step does not happen for anyone else. |
| **Vendoring** | Copying third-party expression into this repo verbatim. Not the approach at any stage. An external skill is adopted only by re-authoring it to the house standard after it has earned its place by use, which is blank-page authoring of examined prior art ([ADR-0005](decisions/0005-author-from-a-blank-page.md)), not a copy. |

## Skills

| Term | Meaning |
|---|---|
| **Skill** | A `SKILL.md` plus optional supporting files. Model-invocable when it keeps a `description`; user-only when it sets `disable-model-invocation: true`. |
| **Shim** | A user-invoked skill whose entire body composes other skills by name, for example "run `/grilling`, using the `/domain-modeling` skill". The cheapest form of composition, and the reason cross-plugin composition works at all. |
| **Composition by name** | Reaching another skill by naming it in prose rather than by including its files. Works across plugin boundaries at runtime, and fails **silently** when the named skill is absent, which is what [ADR-0006](decisions/0006-declare-invoked-skills-as-dependencies.md) exists to prevent. |
| **Workflow** | A multi-step process that spawns agents and coordinates them. `ship` is the only one today. Distinct from a skill, which is a single body of instruction. |
| **Library skill** | A skill invoked only by name from another skill or agent, never a user or model entry point, so it sets `disable-model-invocation: true`. It carries **knowledge**, not a workflow's execution. |
| **Review baseline** | The shared review knowledge: the adversarial stance, the twelve smells, the verify-and-failure-scenario discipline, and the blocking-versus-advisory classification. Decided to live once as a `review-baseline` library skill that every reviewer loads ([ADR-0010](decisions/0010-extract-the-review-baseline-into-a-library-skill.md)), separating it from each reviewer's own execution. |

## Workflow execution

| Term | Meaning |
|---|---|
| **Agent** | A subagent defined in a plugin's `agents/`, namespaced `<plugin>:<name>` (for example `ship:implementer`). Ships inside the plugin that spawns it, never installed loose. |
| **Cold agent** | An agent sharing no conversation with whoever spawned it. Everything it needs is in its brief or fetched by it. The independence is the point, not an accident. |
| **Orchestrator** | The role the `ship` skill plays: resolves the work item, briefs and spawns agents, disposes of findings, decides whether the loop continues. Writes no product code and reviews none. |
| **Repo contract** | The set of facts a cold agent needs to work in a repo without guessing (base, bootstrap, verify, branch, commit, invariants, PR template). Lives in the target repo's `AGENTS.md` under `## Agent contract`. |
| **Round** | One implement-then-review cycle. Capped at three. |
| **Disposition** | The recorded outcome of a review finding: **fix**, **dispute**, or **defer**. Every finding gets exactly one, on the record. |
| **Circuit breaker** | A condition that halts a run at a round boundary (unrunnable verify, runaway diff, a finding recurring with no progress, CI red twice, any force-push). |
| **Preflight and refuse** | The house posture: a workflow needing a tool checks for it and stops with a clear message, rather than degrading to a quieter path that then rots. |
