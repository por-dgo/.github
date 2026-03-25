# por-dgo

Software projects developed at **Teledyne DGO, Portsmouth**. This GitHub organization hosts internal tools, utilities, and automation built by the DGO software team.

This repository (`.github`) contains the organization profile, shared GitHub Copilot instructions, and default templates inherited by all repositories.

---

## Repositories

| Repository | Description | Status |
|------------|-------------|--------|
| [ifs_umbrella](https://github.com/por-dgo/ifs_umbrella) | IFS Cloud development toolkit — shared templates, API discovery tools, and reference documentation | Active |
| [ifs_silver](https://github.com/por-dgo/ifs_silver) | IFS Silver Lining — supervisor dashboard for shop orders, quality tracking, and analytics | Active |
| [ifs_breeze](https://github.com/por-dgo/ifs_breeze) | IFS Breeze — operator portal for shop floor terminals | In development |

---

## Getting Started

### Prerequisites

- **Python 3.13+** — [python.org/downloads](https://www.python.org/downloads/)
- **Git** — [git-scm.com](https://git-scm.com/)
- **GitHub CLI** — [cli.github.com](https://cli.github.com/) (authenticate with `gh auth login`)
- **VS Code** — [code.visualstudio.com](https://code.visualstudio.com/)
- **Windows** — all projects target Windows as the deployment platform

### Clone and Set Up a Project

```powershell
gh repo clone por-dgo/<repo-name>
cd <repo-name>
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
```

Each project's own README contains specific setup notes and configuration details.

---

## Branching and Workflow

- **Default branch**: `main` — always kept in a working state.
- **Feature branches**: Create a branch for any non-trivial change. Name it descriptively (e.g., `add-excel-export`, `fix-pagination-bug`).
- **Merging**: Merge feature branches into `main` when complete. Keep branches after merging — they serve as a visible history of development work.
- **Commit messages**: Freeform but descriptive — summarize *what* changed and *why*. One logical change per commit.

```powershell
git checkout -b add-dashboard-filter
# ... make changes ...
git commit -m "Add part number filter to shop floor dashboard"
git checkout main
git merge add-dashboard-filter
```

---

## Release Process

Releases use **git tags** as the single source of truth for version numbers. No version strings are maintained in source code.

### Quick Reference

```powershell
# 1. Commit your changes
git add -A
git commit -m "v0.17.0: description of what changed"

# 2. Tag the release
git tag v0.17.0

# 3. Push
git push
git push --tags

# 4. Build (from the project directory)
python build.py

# 5. Create a GitHub Release (attaches the .zip)
gh release create v0.17.0 dist/*.zip --title "v0.17.0" --notes "Description of changes"
```

### Distribution

- **GitHub Releases** — primary distribution channel. End users download from the repository's Releases page.
- **Network share** — secondary channel for users who do not have GitHub access.
- **Packaging note**: All releases are distributed as `.zip` files containing the `.exe`. Direct `.exe` downloads may trigger false positives from endpoint protection (Cortex XDR). Always distribute inside a `.zip`.

---

## Copilot and AI-Assisted Development

This organization uses **GitHub Copilot** for AI-assisted development. Copilot instructions are layered:

### Instruction Inheritance

```
Organization (.github/copilot-instructions.md)
  └── Repository (.github/copilot-instructions.md)  ← overrides org defaults
```

- **Org-level instructions** (this repo) define general coding conventions and security guidelines that apply to every project.
- **Repo-level instructions** override or extend the org defaults with project-specific context — API patterns, domain knowledge, architectural decisions.

### For Contributors

When working on a repository, Copilot automatically loads both layers. To add or update project-specific instructions, edit `.github/copilot-instructions.md` in that repository. See [ifs_umbrella](https://github.com/por-dgo/ifs_umbrella) for a detailed example.

---

## Security

- All repositories in this organization are **private**. Do not change repository visibility without approval.
- Do not commit internal URLs, hostnames, IP addresses, credentials, or infrastructure details to any file — including code comments, documentation, and error messages.
- This is an **ITAR/CUI-sensitive environment**. Treat all project data accordingly.
- Desktop tools run on **localhost only** with no external network exposure.
- Authentication relies on Windows domain SSO — no credentials are stored in source code.
