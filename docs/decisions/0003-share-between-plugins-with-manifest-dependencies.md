# 0003. Share between plugins with manifest dependencies

- **Status:** Accepted
- **Date:** 2026-07-25

## Context and problem statement

`AGENTS.md` asserted, without a recorded reason, that "Content shared between
plugins gets duplicated or symlinked, never referenced upward." That rule was
written from the (correct) observation that relative paths out of a plugin root
break after install. It missed that Claude Code supports a `dependencies` field
in `plugin.json`, which references another plugin legally and is managed by the
platform.

## Decision drivers

- Installing copies a plugin into a version-keyed cache directory. Paths that
  traverse outside the plugin root (`../shared/`) resolve to nothing there.
- Symlinks inside a plugin **are** supported: one resolving elsewhere in the same
  marketplace is dereferenced and its content copied in. But on Windows this
  needs `mklink /D` from an elevated prompt or Developer Mode, and under
  `--plugin-dir` or a local-path install every symlink leaving the plugin's own
  directory is skipped. Authoring and installed behaviour would differ.
- The primary development machine is Windows.
- A `dependencies` entry is a **bare plugin name** by default, resolving within
  the declaring plugin's own marketplace. No git tags, no allowlist, no config.
- A **version constraint** changes that: resolution then runs against git tags
  named `{plugin-name}--v{version}`, and a missing tag disables the dependent
  plugin with `no-matching-tag`.
- A dependency installs another **plugin**. It does **not** grant access to that
  plugin's files.

## Considered options

- Manifest `dependencies`
- Symlinks across the marketplace
- Duplicate freely, treat every plugin as standalone
- Relative paths out of the plugin root (not viable, listed for the record)

## Decision outcome

Chosen option: "Manifest `dependencies`", declared by bare name within this
marketplace, because it is real dependency management (auto-install, transitive
enable, blocked disable, `prune`) at zero configuration cost and with no Windows
friction.

Version constraints are deliberately **not** used until a real compatibility
break appears, because they would oblige tagging every plugin release.

Nothing is extracted into a shared plugin until a second consumer actually
exists. The decision is recorded now so it is not re-litigated under pressure
later.

### Consequences

- Good, because sharing is declared, enforced at install and load, and cleaned up
  by `claude plugin uninstall --prune`.
- Good, because it works identically on Windows and elsewhere.
- Bad, because dependencies share **components reachable by name**, not files.
  Content that genuinely must be read as a file still has to live inside one
  plugin, which is why ADR-0002 keeps `ship` whole.
- Bad, because it supersedes the `AGENTS.md` invariant quoted above, which must
  be corrected rather than left contradicting this record.
- Note: cross-marketplace dependencies are possible but blocked by default. They
  need `allowCrossMarketplaceDependenciesOn` in the root marketplace manifest,
  and the consumer must already have added the target marketplace.

## Pros and cons of the options

### Manifest `dependencies`

- Good, because the platform installs, enables, and prunes it for you.
- Good, because bare names need no tags and no allowlist within one marketplace.
- Bad, because it cannot share a file, only a named component.

### Symlinks across the marketplace

- Good, because it does share actual file content, dereferenced into the cache.
- Bad, because creating them on Windows needs elevation or Developer Mode.
- Bad, because local-path and `--plugin-dir` installs skip them, so the authoring
  setup and the installed setup behave differently.

### Duplicate freely

- Good, because it is the simplest thing to reason about.
- Bad, because copies drift, which is the failure the original invariant was
  written to prevent.

### Relative paths out of the plugin root

- Bad, because they do not resolve after install. Recorded so the option is
  visibly closed.
