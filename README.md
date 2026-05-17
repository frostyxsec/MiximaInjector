# Mixima — Android Pentesting Toolkit

Automated APK patching, security bypass, static analysis & universal ReVanced patching.

```
███╗░░░███╗██╗██╗░░██╗██╗███╗░░░███╗░█████╗░
████╗░████║██║╚██╗██╔╝██║████╗░████║██╔══██╗ Toolkit
██╔████╔██║██║░╚███╔╝░██║██╔████╔██║███████║ @Frostyxsec
██║╚██╔╝██║██║░██╔██╗░██║██║╚██╔╝██║██╔══██║
██║░╚═╝░██║██║██╔╝╚██╗██║██║░╚═╝░██║██║░░██║
╚═╝░░░░░╚═╝╚═╝╚═╝░░╚═╝╚═╝╚═╝░░░░░╚═╝╚═╝░░╚═╝
```

---

## Features

### 🔧 APK Injection (Smali Patches)
Bypass security mechanisms via smali-level code injection:

| Method | Description |
|---|---|
| `sslbypass` | Bypass SSL Certificate Pinning |
| `hostnameremove` | Remove Hostname Verification |
| `rootemuremove` | Bypass Root & Emulator Detection |
| `bypassrom` | Spoof device ROM/ID (Telephony, WiFi MAC, Build props, etc.) |
| `sdkremove` | Remove minimum SDK version restriction |
| `removeads` | Strip ad libraries & ad unit IDs |
| `removeblockscreenshot` | Disable screenshot blocking (FLAG_SECURE) |
| `killsignature` | Bypass APK signature verification |
| `bypassbiometric` | Bypass biometric/fingerprint authentication |

### 🌐 ReVanced Universal Patching
Apply universal patches via `revanced-cli` + `patches.rvp`:

| Short Name | Description |
|---|---|
| `exportactivities` | Export all activities |
| `hideadb` | Hide ADB status |
| `spoofbuild` | Spoof device build info |
| `hidemock` | Hide mock location |
| `spoofsim` | Spoof SIM country |
| `spoofwifi` | Spoof Wi-Fi connection |
| `enabledebug` | Enable Android debugging |
| `exportdata` | Export internal data documents provider |
| `overridessl` | Override certificate pinning |
| `removescreenblock` | Remove screen capture restriction |
| `removescreenshot` | Remove screenshot restriction |
| `removeshare` | Remove share targets |
| `disablepairip` | Disable Pairip license check |

### 🔍 SAST — Static Application Security Testing
Analyze decompiled APK for security vulnerabilities:

- **Manifest analysis** — permissions, exported components, debug flags, cleartext traffic
- **Source code analysis** — hardcoded secrets, weak crypto, SSL issues, WebView risks
- **Native library inspection** — debug symbols, text relocations
- **Resources analysis** — sensitive strings, URLs, API keys
- **Risk assessment** — high/medium/low severity scoring
- **SAST templates** — 40+ YAML-based pattern detectors (AWS keys, Firebase, Stripe, etc.)
- **Output** — plaintext reports & comprehensive HTML reports

---

## Requirements

### Core
- **apktool** — APK decompilation & recompilation
- **Java JDK 17+** — `keytool` for keystore, `jarsigner` for APK signing
- **aapt** — APK info extraction (SAST)
- **zip/unzip**, **openssl**, **file**, **binutils**

### Optional
- **apksigner** (Android SDK build-tools) — V2/V3 signature scheme support
- **zipalign** (Android SDK build-tools) — APK optimization
- **jadx** — Java source decompilation (SAST)
- **revanced-cli** — ReVanced universal patching

### Install Dependencies

```bash
# Debian/Ubuntu
sudo apt update
sudo apt install apktool openjdk-17-jdk aapt zip unzip openssl jadx file binutils

# revanced-cli (optional — for ReVanced patching)
mkdir -p ~/revanced-cli
wget -O ~/revanced-cli/revanced-cli-6.0.0-all.jar \
  https://github.com/revanced/revanced-cli/releases/download/v6.0.0/revanced-cli-6.0.0-all.jar

# Create wrapper
cat > ~/.local/bin/revanced-cli << 'EOF'
#!/bin/bash
JAR="$HOME/revanced-cli/revanced-cli-6.0.0-all.jar"
exec java -jar "$JAR" "$@"
EOF
chmod +x ~/.local/bin/revanced-cli
```

---

## Usage

### Basic Injection

```bash
./Mixima --apk app.apk --output patched.apk --injection sslbypass,rootemuremove
```

### Multiple Methods

```bash
./Mixima --apk secure.apk --output bypassed.apk \
  --injection sslbypass,killsignature,bypassbiometric
```

### Resource Fix Mode
For APKs that fail to decompile due to broken resources:

```bash
./Mixima --apk problematic.apk --output fixed.apk \
  --injection sslbypass --fix-res
```

### ReVanced Universal Patching

```bash
# All available patches
./Mixima --apk app.apk --output universal.apk --revanced

# Specific patches
./Mixima --apk app.apk --output patched.apk --revanced \
  --revanced-methods "enabledebug,overridessl,exportactivities"
```

### Combine Injection + ReVanced

```bash
./Mixima --apk app.apk --output full.apk \
  --injection sslbypass,killsignature --revanced
```

### SAST Static Analysis

```bash
# SAST only (no patching)
./Mixima --apk app.apk --sast

# SAST with text report
./Mixima --apk app.apk --sast --sast-output results.txt

# SAST with HTML report
./Mixima --apk app.apk --sast --sast-output-html report.html

# SAST + patching
./Mixima --apk app.apk --output patched.apk \
  --injection sslbypass --sast --sast-output scan.txt
```

### List Available Methods

```bash
./Mixima --help
./Mixima --list-revanced-methods
```

---

## Directory Structure

```
Mixima/
├── Mixima                  # Main script (bash)
├── MiximaSAST.sh           # Standalone SAST analysis script
├── patches.rvp             # ReVanced universal patches bundle
├── mixima_keystore.keystore # Auto-generated signing keystore
├── Addons/
│   ├── BypassRom/
│   │   └── DeviceID.smali  # Device spoofing smali payload
│   ├── KillSignature/
│   │   └── Fix.smali       # Signature bypass smali payload
│   └── BypassBiometric/
│       └── BiometricBypass.smali  # Biometric bypass smali payload
└── MiximaSAST/
    ├── Android/             # Android-specific SAST YAML templates
    └── Keys/                # API key/secret SAST YAML templates
```

---

## How It Works

1. **Decompile** — APK is decompiled with `apktool`
2. **Inject** — Smali-level patches are applied based on selected methods
3. **Recompile** — Modified APK is recompiled (with aapt2 fallback)
4. **Sign** — APK is signed with auto-generated keystore (V2/V3 schemes)
5. **Optional: SAST** — Static analysis runs on decompiled code after Step 1
6. **Optional: ReVanced** — ReVanced CLI patches applied after signing

---

## Notes

- Patched APKs are for **authorized security testing only**
- Always test on a dedicated device or emulator
- Some APKs may require framework files — run `apktool if` first
- The keystore is auto-generated on first run (`mixima_keystore.keystore`)
- ReVanced mode requires `revanced-cli` + `patches.rvp` (included)

---

## Credits

**@Frostyxsec** — Original Mixima Toolkit
ReVanced team — Universal patches & revanced-cli
