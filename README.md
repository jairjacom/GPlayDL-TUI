# GPlayDL-TUI

<div align="center">
    <img src="https://raw.githubusercontent.com/jairjacom/GPlayDL-TUI/main/.images/Gplaydl_background.jpg" width="350px">
</div>


**A feature-rich Terminal UI for downloading Android APKs from Google Play**

*Built for Termux · Powered by gplaydl*

[![Python](https://img.shields.io/badge/Python-3.14.4%2B-blue?style=for-the-badge&logo=python&logoColor=white)](https://python.org)
[![Platform](https://img.shields.io/badge/Platform-Termux-green?style=for-the-badge&logo=android&logoColor=white)](https://termux.dev)
[![License](https://img.shields.io/badge/License-MIT-purple?style=for-the-badge)](LICENSE)
</div>

---

## ✨ Features

| Feature | Description |
|---------|-------------|
| 🔍 **Search & Download** | Search Google Play and download any app by name |
| 📦 **Split APK Merge** | Automatically merges split APKs into a single installable APK using APKEditor |
| ✍️ **APK Signing** | Sign merged APKs with your keystore using apksigner (V1 + V2 + V3) |
| 📱 **Device Profile** | Replace the device profile used for downloads with any custom device |
| 📲 **Auto Install** | Automatically open the Android package installer after download |
| ⚙️ **Full Configuration** | Persistent config for all settings saved between sessions |
| ☕ **Auto JDK Setup** | Detects and installs openjdk-21 automatically if missing |
| 🔧 **Auto jq Setup** | Detects and installs jq automatically if missing |
| 📦 **Auto gplaydl Setup** | Installs and updates gplaydl via pip automatically |
| 🎨 **Beautiful TUI** | Fully coloured terminal interface with progress bars and status indicators |

---

## 📋 Requirements

### Termux Packages

```bash
pkg update && pkg upgrade
pkg install python git jq openjdk-21 -y
pip install gplaydl 
```

---

### ⚙️ Manual Dependencies (Optional)

The `bin/` folder lives next to `gplaydl_tui.py` — wherever you clone/run the
script from is where it will look for and create `bin/`. Place `apksigner.jar`
there before first run if you want signing support.

| File | Purpose | Source |
|---------|-------------|--|
| APKEditor.jar | Merges split APKs | Auto-downloaded from [REAndroid/APKEditor](https://github.com/REAndroid/APKEditor) |
| apksigner.jar | Signs Apks | already present and can be obtained from Build Tools |

---

### 🚀 Installation

# 1. Clone the repository
```
git clone https://github.com/jairjacom/GPlayDL-TUI.git
cd GPlayDL-TUI
```

# 2. Make the script executable
```
chmod +x gplaydl_tui.py
```

# 3. Run it
```
python gplaydl_tui.py
```

## 📋 Project Structure

All of this lives next to `gplaydl_tui.py`, wherever you cloned it:

```
GPlayDL-TUI/                  ← the folder you cloned/run the script from
├── gplaydl_tui.py
├── bin/
│   ├── APKEditor.jar        ← auto-downloaded on first run
│   └── apksigner.jar        ← place manually for signing support
├── .config/
│   └── gplaydl-tui/
│       └── config.json      ← your saved settings
└── <package.name>/          ← temp download/merge folder (auto-cleaned)

~/.config/gplaydl/
└── auth_arm64.json          ← gplaydl authentication & device profile (gplaydl default fetched)
```

Downloaded APKs go to your configured output directory (default `~/gplay`,
set via Configure in the menu) — that's separate from the install folder above.

## 📖 Usage

```
  ═══════════════════════════════════════════════════════════════
  ● SPLIT   ● SKIP-EXTRAS   ○ SIGN   ○ INSTALL   Arch: arm64 (default)
  ═══════════════════════════════════════════════════════════════

   1   🔍  Search & Download
   2   📱  Replace Device Profile
   3   ⚙   Configure
   4   🔄  Force Re-Authenticate
   5   👤  Link Personal Account
   6   🚪  Exit

  ───────────────────────────────────────────────────────────────
                      Created by GrayWizard
  ───────────────────────────────────────────────────────────────
```

# 1️⃣ Search & Download

1. Enter an app name to search Google Play
2. Select the app number from the results table
3. Review app info and confirm the download
4. The script handles the rest automatically

# 2️⃣ Replace Device Profile

Spoof the device identity used when communicating with Google Play.
Useful for downloading APKs targeting specific hardware or Android versions.

Accepted JSON input formats:

```
// ── Format 1 – Full auth.json style ──────────────────────────
{
  "deviceInfoProvider": {
    "authUserAgentString": "GoogleAuth/1.4 (Pixel 9a BD4A.250405.003)",
    "sdkVersion": "35",
    "properties": {
      "Build.MODEL": "Pixel 9a",
      "Build.MANUFACTURER": "Google"
    }
  }
}
```
```
// ── Format 2 – Bare provider block ───────────────────────────
{
  "authUserAgentString": "GoogleAuth/1.4 (Pixel 9a BD4A.250405.003)",
  "sdkVersion": "35",
  "properties": {
    "Build.MODEL": "Pixel 9a",
    "Build.MANUFACTURER": "Google"
  }
}
```
```
// ── Format 3 – Partial snippet (no outer braces needed) ──────
"deviceInfoProvider": {
  "authUserAgentString": "GoogleAuth/1.4 (Pixel 9a BD4A.250405.003)",
  "sdkVersion": "35",
  "properties": {
    "Build.MODEL": "Pixel 9a"
  }
}
```
> "⚠️ A backup of your original auth file is always saved as
auth_arm64.json.bak before any changes are applied."

# 3️⃣ Configure

| Option | Settings | Description |
|---------|-------------|----|
| 1 | Prefer Split APKs | ON = download splits and merge · OFF = --no-splits |
| 2 | Output Directory | Where to save the final APK (default: $HOME/GPlayDL-TUI) |
| 3 | Dispenser Link | Custom token dispenser URL |
| 4 | Skip Extras | ON = --skip-extras to skip OBB / patch files (recommended to off during downloading games) |
| 5 | Architecture | arm64 · armv7 · or let gplaydl decide (default) |
| 6 | Keystore | Path to .jks / .p12 / .pfx keystore file |
| 7 | Sign APKs | Toggle APK signing on / off (requires keystore) |
| 8 | Auto Install APK | Open the Android installer automatically after download/sign |
| 9 | Personal Account | Email of a linked gplaydl account (set via Link Personal Account, or edit here) |
| 10 | Use Personal Acct | Toggle whether searches/downloads use the linked account (ON) or the default shared pool (OFF) |
| 11 | Save & Return | save all changes to .config |
| 0 | Discard & Return | Abandon all unsaved changes |

## 🔑 APK Signing

| Scheme | Status
|---------|-------------|
| V1 – JAR Signing | ✅ Enabled ( Required for android 1.0+) |
| V2 – APK scheme Signing | ✅ Enabled ( Required for Android 7.0+) |
| V3 – APK scheme Signing | ✅ Enabled ( Required for Android 9.0+) |

---

## 📱 Device Profile Spoofing

You can make Google Play think you are downloading from a completely
different device. Common use cases:

1) App is not available for your device

2) Download a specific ABI variant (arm64 vs armv7)

3) Target a specific SDK / Android version

4) Bypass some device compatibility restrictions

The script uses jq to surgically replace only the
deviceInfoProvider block inside auth_arm64.json — all tokens,
cookies and credentials remain untouched.

---

## 👤 Personal Account Linking

By default, gplaydl uses a shared anonymous token pool for downloads,
which can hit rate limits. As of gplaydl 4.x, you can link your own
Google account instead — the TUI has a guided menu option for this.

> ⚠️ Google may flag or lock accounts used with unofficial clients.
> Use a spare/burner Google account, never your primary one.

**Setup (Main Menu → 5. Link Personal Account):**

1. Install the [gplaydl Authenticator](https://github.com/rehmatworks/gplaydl-authenticator) app on any Android phone.
2. Sign in with a spare Google account.
3. Open **"Link gplaydl"** in the app to get a one-time pairing code.
4. Back in the TUI, enter the pairing code when prompted — it runs
   `gplaydl link --code XXXX` for you.
5. Enter the email you signed in with, so the TUI can pass it to
   gplaydl (`--email`) and select that account for downloads.

This only affects your own device — the linked account and API key
are stored locally at `~/.config/gplaydl/config.json` and are never
part of this repository. Cloning this fork does not give anyone
access to your linked account.

To stop using a personal account, clear the email in
**Configure → 9. Personal Account** (falls back to the default pool).

---

## ⚙️ Configuration File

All settings are persisted at $HOME/gplay/.config/gplaydl-tui/config.json:

```
{
  "prefer_split"  : "on",
  "output_dir"    : "/sdcard/APKs",
  "dispenser_link": "",
  "account_email" : "",
  "skip_extras"   : "on",
  "arch"          : "",
  "keystore_path" : "/path/to/my.jks",
  "keystore_alias": "mykey",
  "keystore_pass" : "••••••••",
  "key_pass"      : "",
  "keystore_type" : "JKS",
  "sign_apk"      : "on",
  "auto_install"  : "off"
}
```
---

<div align="center">
Made with ❤️ by GrayWizard

If this tool helped you, consider giving it a ⭐ on GitHub

</div> 



