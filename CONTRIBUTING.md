# Contributing to por-dgo Projects

This guide covers the standard workflow for all repositories in the **por-dgo** organization.

## Issues

Every feature, bug fix, or chore starts as a GitHub Issue. Use the issue templates when creating new issues:

- **Feature Request** — new functionality or improvements
- **Bug Report** — something broken or behaving unexpectedly
- **Task** — refactors, documentation, build changes, chores
- **New App Idea** — propose a new IFS Cloud workflow tool

Add appropriate **labels** (see below) and assign to a **milestone** if the target release is known.

## Branches

Create a branch for every non-trivial change. Use the "Create a branch" button on the issue page, or:

```
git checkout -b {issue#}-{short-description}
```

Examples:
- `12-add-csv-export`
- `34-fix-login-timeout`
- `7-update-readme`

## Pull Requests

Open a PR when work is ready. The PR template will prompt you to:

1. Describe **what** changed
2. Link the issue with `Closes #12` (auto-closes on merge)
3. Confirm testing

All repos use **squash merge** — your PR commits get combined into a single clean commit on `main`. Feature branches are **automatically deleted** after merge.

Self-review your own diff before merging — the GitHub diff view often reveals things your editor doesn't.

## Sprint Rhythm

| When | What |
|------|------|
| **Monday** | Review backlog, pick 2–4 issues for the week, move to Sprint column |
| **During week** | Work issues: branch → commit → PR → merge |
| **Friday** | Tag a release if enough is done, move leftovers back to Backlog |
| **Monthly** | Groom backlog — close stale issues, reprioritize |

## Releases

Releases use **git tags** as the single source of truth for version numbers.

```powershell
# 1. Update CHANGELOG.md
# 2. Commit
git commit -m "v1.2.0: description of what changed"

# 3. Tag
git tag v1.2.0

# 4. Push
git push && git push --tags

# 5. Build and create GitHub Release
python build.py
gh release create v1.2.0 dist/*.zip --title "v1.2.0" --notes "Description"
```

## Labels

Standard labels used across all repos:

| Label | Purpose |
|-------|---------|
| `bug` | Something is broken |
| `enhancement` | New feature (auto-applied by feature template) |
| `task` | Chore, refactor, docs |
| `idea` | New app proposal |
| `infrastructure` | Build, packaging, auth, config |
| `priority: high` | Do this week |
| `priority: med` | Do this sprint/next |
| `priority: low` | Backlog |
| `shop-floor` | Shop Floor dashboard tab |
| `quality` | Quality/MRB tab |
| `history` | History/Analytics tab |
| `api` | IFS Cloud API integration |
| `blocked` | Waiting on something external |
| `needs-info` | Needs more detail before work can begin |

## Commit Messages

Freeform but descriptive — summarize *what* changed and *why*. Reference issue numbers where relevant.

```
Add CSV export option (#12)
Fix timeout on large queries
v1.2.0: dashboard filtering + performance improvements
```
