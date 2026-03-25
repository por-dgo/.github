# Teledyne DGO — Copilot Instructions

These instructions apply to all repositories in the **por-dgo** organization unless overridden by a repo-level `copilot-instructions.md`.

## Organization Context

- **Organization**: Teledyne DGO (named for founder Donald G. O'Brien), a Teledyne Marine company
- **Site**: Portsmouth, NH (headquarters: 162 Corporate Drive, Portsmouth, NH 03801)
- **Specialty**: High-reliability electrical interconnect systems — glass-to-metal sealed connectors, penetrators, cable assemblies, and feedthrough systems for extreme environments (subsea, nuclear, defense, aerospace)
- **Primary system**: IFS Cloud 24R2 (enterprise ERP by IFS AB)
- **Base URL**: `https://us3-vpifs-mid01.tdy.teledyne.com/main/ifsapplications`

## Tech Stack Defaults

| Layer | Technology |
|-------|-----------|
| Backend | Python 3.13+, Flask 3.1 |
| Frontend | Vanilla JS, CSS custom properties, Jinja2 templates |
| Auth | Playwright browser context (NTLM/Kerberos SSO) or OAuth 2.0 (IAM) |
| API | OData v4 REST via IFS projection gateway |
| Packaging | PyInstaller → single-file `.exe` |
| Settings | `%APPDATA%/{AppName}/settings.json` |

## IFS Cloud API Patterns

### Gateway Pattern
- API: `/main/ifsapplications/projection/v1/{ProjectionName}.svc/{EntitySet}`
- Web client: `/main/ifsapplications/web/page/{PageName}/{Form};$filter={filter}`
- Auth realm (Keycloak): `tissprod`
- Site: `TDGO`, Company: `25`

### OData Query Rules
- **NEVER use urllib.parse.urlencode()** — it mangles the `$` prefix on OData params
- Build query strings manually: `f"?$filter={expr}&$top=50"`
- **Avoid $select** — many IFS projections return errors when $select is used; omit to get all fields
- String values: single quotes in `$filter` → `OrderNo eq '1071168'`
- Integer values: no quotes → `OpId eq 12345`
- Date values: ISO 8601 → `DtReleased ge 2026-02-01`
- Enum values: fully qualified → `Objstate eq IfsApp.DocumentRevisionsHandling.DocIssueState'Released'`

### Required Headers
```
Accept: application/json;odata.metadata=full;IEEE754Compatible=true
```
- `IEEE754Compatible=true` is required for wizard fields (string-encoded numbers)
- `verify=False` for self-signed corporate certificates

### Objstate Is Not Filterable
On our IFS instance, filtering by `Objstate` in OData `$filter` always returns HTTP 400. **Workaround**: query by date range, then filter by status in Python.

### Pagination
Large result sets return `@odata.nextLink`. Always follow pagination links — never assume a single response has all data.

### Virtual Entity Wizard Pattern (Multi-Step Mutations)
IFS wizards are NOT simple POST/PUT — they require multiple steps:
1. **POST** `/{EntitySet}` — create virtual entity (returns `Objkey` + `@odata.etag`)
2. **PATCH** `/{EntitySet}(Objkey='...')` — set field values (pass `If-Match` etag)
3. **POST** `.../{Action}_FinishExecute` — execute the operation
4. **POST** `.../{Action}_CleanupVirtualEntity` — **always** cleanup, even on failure

### Bound Actions (Simpler)
```
POST /{EntitySet}(Key=Value)/IfsApp.{Projection}.{Type}_{Action}
Body: {}
```

## Authentication Architecture

### Critical: IFS Session Binding
IFS Cloud binds sessions to the browser's TCP/TLS connection. Extracting cookies and replaying them with `requests` **does not work**. Keep a Playwright browser context alive and route ALL API calls through `context.request()`.

### Thread Safety
Flask runs on multiple threads; Playwright's event loop is single-threaded. Direct Playwright calls from Flask threads **will deadlock**. Use a request queue + `threading.Event` pattern: Flask threads enqueue requests and wait for results.

### XSRF Tokens
IFS Cloud requires XSRF tokens for any write operations. Capture these from page loads.

## Code Conventions

- **Naming**: `snake_case` functions/variables, `PascalCase` classes, `UPPER_SNAKE_CASE` constants
- **Logging**: `logging.getLogger(__name__)` per module
- **Error handling**: Custom `IFSError(message, status_code, odata_error)`. Graceful degradation — return `None` or empty list on timeout rather than crashing.
- **Config**: Connection details in `config.py`, user preferences in `settings.py`
- **Versioning**: git tags are the single source of truth (e.g. `v0.16.0`). No version strings in source.
- **Build**: `build.py` generates splash PNG + runs PyInstaller. Entry point: `main.py`.
- **Single-instance**: Windows mutex (`Global\{AppName}`) prevents duplicate processes

## Key Projections

| Projection | Entity Set | Purpose |
|-----------|-----------|---------|
| `ShopOrd.svc` | `ShopOrderSet` | Shop orders (240+ fields) |
| `ShopOrdUtil.svc` | `ShopOrderOperationSet` | Operations on shop orders |
| `ShopOrderOperationsHandling.svc` | `ShopOrderOperationsSet` | Operation state changes |
| `MRBCasesHandling.svc` | `MrbHeadSet` | MRB cases |

## Security Notes

- All apps run **locally** (localhost only, no external exposure)
- No credentials stored — SSO via Windows domain
- ITAR/CUI sensitive environment — no data leaves the network
- Self-signed certs on internal IFS instance
