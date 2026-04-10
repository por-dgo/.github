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

## Python venv Setup — pip/WMI Hang Fix (Windows)

On corporate Windows machines, **Python 3.13 + pip 26** can hang indefinitely on `pip install` due to a WMI deadlock. pip 26 eagerly imports `truststore`, which calls `platform._wmi_query()` — a COM/WMI call that deadlocks when WMI is restricted by Group Policy.

**Quick diagnostic** — if this hangs, you have the issue:
```powershell
python -c "import platform; print(platform.system())"
```

**Fix** — create `.venv/Lib/site-packages/sitecustomize.py` that monkey-patches the WMI call:

```python
import sys

if sys.platform == "win32":
    import platform

    def _wmi_query_bypass(table, *keys):
        if table == "OS":
            ver = sys.getwindowsversion()
            data = {
                "Version": f"{ver.major}.{ver.minor}.{ver.build}",
                "ProductType": ver.product_type,
                "Caption": f"Microsoft Windows {ver.major}.{ver.minor}.{ver.build}",
                "ServicePackMajorVersion": ver.service_pack_major,
                "ServicePackMinorVersion": ver.service_pack_minor,
            }
        elif table == "CPU":
            import struct
            data = {
                "Manufacturer": "GenuineIntel",
                "Caption": f"{struct.calcsize('P') * 8}-bit processor",
            }
        else:
            data = {k: "" for k in keys}
        return tuple(data.get(k, "") for k in keys)

    if hasattr(platform, "_wmi_query"):
        platform._wmi_query = _wmi_query_bypass
```

**Full setup on affected machines:**
```powershell
# 1. Create venv without pip (ensurepip can also hang)
python -m venv .venv --without-pip

# 2. Activate
.\.venv\Scripts\Activate.ps1

# 3. Create the sitecustomize.py patch (paste content above into the file)
# 4. Bootstrap pip — now safe with WMI patched
python -m ensurepip --upgrade

# 5. Install dependencies
pip install -r requirements.txt
```

**Notes:**
- `sitecustomize.py` lives inside `.venv/` (gitignored) — recreate it after making a new venv
- Per-venv fix — apply to each project on affected machines
- For PyInstaller builds, also add the patch as a runtime hook (`rthook_wmi.py`)
- Upstream issue: Python 3.13 regression — may be fixed in a future CPython release
