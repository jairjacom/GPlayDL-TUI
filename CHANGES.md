# GPlayDL-TUI — Changes Log

## Device Profile Auto-Restore Fix
**Date:** 2026-06-17

---

### Problem

`gplaydl` internally resets the `deviceInfoProvider` field inside
`~/.config/gplaydl/auth-arm64.json` every time an auth token expires and
refreshes. This wiped out any custom device profile (e.g. Pixel 9 Pro XL)
silently mid-session, reverting it to gplaydl's built-in default. There was
no hook point to intercept the refresh from outside the tool.

---

### Changes Made to `gplaydl_tui.py`

#### 1. New config key — `device_profile_path`

```python
DEFAULT_CONFIG = {
    ...
    "device_profile_path": "",   # empty = disabled, no behaviour change
}
```

Stores the absolute path to the user's custom device JSON file.
Empty string means the feature is off and nothing changes for existing users.

---

#### 2. New function — `reapply_device_profile(cfg)`

```python
def reapply_device_profile(cfg: dict) -> bool:
```

- Reads `device_profile_path` from config; silently returns `False` if not set
  or if the file does not exist.
- Validates the JSON with the existing `_validate_device_json()` helper.
- Uses `jq` to inject the `deviceInfoProvider` block back into
  `auth-arm64.json` (same atomic write pattern as `do_replace_device_profile`).
- Prints a single confirmation line: `Device profile restored → <Model>`.
- Returns `True` on success, `False` on any failure (never raises).

---

#### 3. Auto-restore calls after every gplaydl command

Three locations in the script now call `reapply_device_profile(cfg)` right
after each gplaydl subprocess exits:

| Function | Where |
|---|---|
| `do_search_download()` | after the `search` command |
| `do_search_download()` | after the `download` command |
| `do_force_reauth()` | after the fresh `auth` command |

This ensures that even if a token refresh happened inside the gplaydl call,
the correct device profile is restored before anything else runs.

---

#### 4. `do_replace_device_profile()` updated signature and saves the path

The function now accepts and returns `cfg`:

```python
def do_replace_device_profile(cfg: dict) -> dict:
```

After a successful profile apply it does:

```python
cfg["device_profile_path"] = json_path
save_config(cfg)
```

And `main_menu` is updated to pass and capture `cfg`:

```python
cfg = do_replace_device_profile(cfg)
```

So the user only needs to set the profile once via **Menu → 2 (Replace Device
Profile)**. From that point on, auto-restore is active for every subsequent
search, download, and re-auth operation without any extra steps.

---

#### 5. Minor UI fix in `do_configure`

Output Directory display when not set changed from:

```
(not set)
```

to:

```
(not set – uses ~/gplay)
```

---

### How to Use

1. Launch the TUI and go to **Menu → 2 (Replace Device Profile)**.
2. Enter the path to your device JSON file (e.g. `~/GPlayDL-TUI/auth.json`).
3. Confirm the apply prompt.
4. Done — the path is saved and auto-restore activates silently for all future
   search, download, and re-auth operations.

### How to Disable

Set `device_profile_path` to `""` in:

```
~/<project-dir>/.config/gplaydl-tui/config.json
```

---

### Device Profile Used (Pixel 9 Pro XL)

File: `~/GPlayDL-TUI/auth.json`

| Field | Value |
|---|---|
| Model | Pixel 9 Pro XL (komodo) |
| Build ID | CP31.260522.006 |
| Fingerprint | google/komodo_beta/komodo:CinnamonBun/CP31.260522.006/... |
| SDK | 37 (Android 17 — CinnamonBun) |
| Security patch | 2026-05-05 |
| ABI | arm64-v8a |

Values were pulled live from the device via `getprop`.
