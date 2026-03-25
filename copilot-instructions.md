# Teledyne DGO — Copilot Instructions

These instructions apply to all repositories in the **por-dgo** organization unless overridden by a repo-level `copilot-instructions.md`.

## Organization Context

- **Organization**: Teledyne DGO, a Teledyne Marine company
- **Site**: Portsmouth, NH
- **Target platform**: Windows (primary deployment target for all tools)

## Code Conventions

- **Naming**: `snake_case` functions/variables, `PascalCase` classes, `UPPER_SNAKE_CASE` constants
- **Logging**: `logging.getLogger(__name__)` per module
- **Error handling**: Graceful degradation — return `None` or empty list on timeout rather than crashing
- **Versioning**: git tags are the single source of truth (e.g. `v0.16.0`). No version strings in source code.

## Security

- ITAR/CUI sensitive environment — do not expose internal URLs, hostnames, credentials, or infrastructure details in code comments, logs, or error messages
- All desktop tools run locally (localhost only, no external network exposure)
- No credentials stored in source — use SSO or environment-provided auth
