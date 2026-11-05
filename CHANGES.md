# GPlayDL-TUI — Changes Log

## Android 16+ / SDK 36+ Device Profile Support
**Date:** 2026-11-05

---

### Problem

All gplaydl bundled device profiles max out at SDK 35 (Android 15), so apps
requiring Android 16+ get served old versions. A plain `deviceInfoProvider`
swap in the auth file only changes the User-Agent claim — the real fix is to
POST the profile's raw properties dict to the dispenser's `/api/auth` endpoint,
which returns a complete auth dict including a matching `gsfId` (a genuine
device identity registered at checkin).

---

### Changes

#### `profiles/komodo_sdk37_arm64.properties` (new)

A bundled SDK 37 (Android 17) device profile captured from a real Pixel 9
Pro XL (`komodo`): `Build.VERSION.SDK_INT=37`, `Build.VERSION.RELEASE=17`,
`Platforms=arm64-v8a`, plus the host-captured Features (164) and
SharedLibraries (50). The SPDX/Calyx GPL-3.0 header is retained.

#### `gplaydl_tui.py`

- New helpers: `_load_properties_file`, `properties_to_provider`,
  `list_bundled_profiles`, `dispenser_link`, `mint_token_for_profile`.
- `_validate_device_json` renamed to `_load_device_provider`; it now converts
  `.properties` files into a 5-key provider, mirroring gplaydl's
  `Android-Finsky/…` User-Agent format.
- `reapply_device_profile` now skips rewriting when the fingerprint already
  matches (protects a dispenser-minted provider), creates a one-time `.bak`,
  and reports the SDK.
- `do_replace_device_profile` is now a picker: numbered bundled profiles
  (`◀ active` marker), `[B]` browse custom `.json`/`.properties`, `[M]` mint,
  `[0]` cancel.
- New `[M]` mint flow POSTs the profile to the linked dispenser, validates the
  response before touching disk, persists atomically with `0600` perms, and
  saves `device_profile_minted=on`.
- `status_bar` shows `Dev: <model> · SDK <n>` (green when ≥ 36, yellow below).
- Main menu item 2 relabelled **Device Profile**.
- `do_force_reauth` warns when a re-mint is needed after gplaydl's refresh.

#### `README.md`

Documented bundled profiles, `.properties` custom input, the `[M]` mint
option, and known limitations (50-minute auto-refresh reverts profiles; the
retry chain walks DOWN SDK versions).

---

### Verification

- `python -m py_compile gplaydl_tui.py` — passes.
- Theme smoke test (`apply_theme` over all themes) — passes.
- Key parity vs `gplaydl.profiles._ALL['Pv']['profile']` — no missing/extra keys.
- Converter contract — UA contains `sdk=37`, `platformVersionRelease=17`,
  `supportedAbis=arm64-v8a`.
- Synthetic jq merge — tokens survive the provider swap (`"37"`,
  `"Pixel 9 Pro XL"`, `"DUMMY"`).
- Boot render smoke tests — renders without side effects.

## Color Themes
**Date:** 2026-11-04

---

### Problem

The TUI used a single hardcoded ANSI palette, which was hard to read on
some Termux themes/backgrounds and could not be changed by the user.

---

### Changes Made to `gplaydl_tui.py`

#### 1. New `THEMES` dictionary

Added `THEMES` next to the `C` color class with six selectable palettes:
`default`, `gruvbox-dark`, `moonfly`, `miasma`, `srcery`, and
`posterpole-gray`. Non-default themes use 24-bit truecolor escapes
(`\033[38;2;r;g;bm` for foreground, `\033[48;2;r;g;bm` for background).

#### 2. New `apply_theme(name)` function

Looks up `THEMES` case-insensitively (and forgivingly), then `setattr`s each
mapped attribute on `C`. Unknown names fall back to `"default"`, which
restores every mapped color to its original hardcoded value.

#### 3. New config key — `theme`

```python
DEFAULT_CONFIG = {
    ...
    "theme": "default",
}
```

Persisted exactly like the other settings via Configure → Save & Return.

#### 4. Configure option **11 · Theme**

`do_configure` gained a Theme picker submenu (option 11). It lists every
theme with a live preview sample (`✔ ok  ⚠ warn  ✖ err  ◆ step`) rendered in
that theme's colors using the theme dict directly (the global palette is not
touched for previews). Selecting a theme applies it immediately, so the
redrawn banner/menu uses the new colors right away. Save & Return moved
from option 11 to option 12 (Discard remains 0).

#### 5. Theme applied on startup

`main()` now calls `apply_theme(cfg.get("theme", "default"))` right after
`load_config()` so the first banner renders with the saved theme.

---

### How to Use

1. Launch the TUI and go to **Menu → 3 (Configure)**.
2. Choose **11 · Theme**.
3. Pick a palette from the numbered list — the preview shows each theme live.
4. Choose **12 · Save & Return** to persist the theme.

### How to Disable / Reset

Select the `default` theme from the same menu (it restores the original
built-in ANSI colors).

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
