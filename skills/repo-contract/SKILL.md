---
name: repo-contract
description: "Resolve the repo contract a cold agent needs: base, bootstrap, verify, branch, commit, invariants, PR template. Use before briefing any cold agent."
---

The contract is everything a cold agent needs in order to work in this repo without guessing. It lives in `AGENTS.md` under a `## Agent contract` heading, so it is version controlled, readable by humans, and correctable once rather than re-guessed every run.

## If the contract exists

Read it and go. Paste it verbatim into every agent brief.

## If it does not

Probe for each field, then confirm before use.

| Field | Where to look |
| --- | --- |
| base | `gh repo view --json defaultBranchRef` |
| bootstrap | the install step in `.github/workflows/*.yml` first, since that is what actually works from a clean checkout. Cross-check against the lockfile and manifest (`uv.lock` + `pyproject.toml`, `package-lock.json` + `package.json`, `Cargo.toml`, `go.mod`) |
| verify | the commands CI runs, read from `.github/workflows/*.yml`. These are authoritative: they are what will gate the PR. Fall back to the manifest's script block only when there is no CI |
| branch | `AGENTS.md`, `CONTRIBUTING.md`, or infer from `git branch -a` |
| commit | `AGENTS.md`, `CONTRIBUTING.md`, `.gitmessage`, or infer from `git log --oneline -30` |
| invariants | `AGENTS.md`, `CLAUDE.md`, `docs/ARCHITECTURE.md`, `docs/decisions/` |
| pr template | `.github/PULL_REQUEST_TEMPLATE.md` |

Then show the user the whole proposed contract, ask them to correct it, and write it into `AGENTS.md` on approval. Create a thin `AGENTS.md` if the repo has none.

Do not run on an unconfirmed contract. Every round inherits its errors, and a wrong verify command costs a full cycle before anything notices.

## Shape

```markdown
## Agent contract

- **base**: main
- **bootstrap**: `uv sync --all-extras --dev --frozen`
- **verify**: `uv run ruff check .`, `uv run ruff format --check .`, `uv run mypy`, `uv run pytest`
- **branch**: `<type>/<issue#>-<slug>`
- **commit**: Conventional Commits with a seam scope, issue number in the subject
- **invariants**: AGENTS.md section "Invariants", docs/ARCHITECTURE.md
- **pr template**: `.github/PULL_REQUEST_TEMPLATE.md`
```

A repo whose conventions are already written down (invariants, a PR template, a documented branch scheme) needs almost nothing from this step. A repo with none of that is where the contract earns its keep, and where confirming it with the user matters most.
