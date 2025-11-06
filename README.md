# `wg-conf-to-mobileconfig`

**Convert a WireGuard `.conf` file into a signed macOS `.mobileconfig` profile** — perfect for deploying via **Jamf Pro**, **Intune**, or manual installation.

---

# Features

- Converts any standard WireGuard `.conf` to a valid `com.apple.vpn.managed` profile  
- Automatically escapes XML special characters  
- Generates unique UUIDs and identifiers per profile  
- Signs with **any certificate** in your macOS keychain  
- Fully **customer-agnostic** — no hard-coded org or cert names  
- Works with **Jamf Pro**, **Apple Configurator**, or `profiles` CLI  

# Requirements

- **macOS** (tested on macOS 14 Sonoma & 15 Sequoia)
- `uuidgen`, `plutil`, `security` (built-in)
- **Signing certificate** with private key in **System** or **login** keychain
- WireGuard app installed on target Macs (for `com.wireguard.macos`)

# Usage

```bash
./wg-conf-to-mobileconfig <config.conf> [output_name] --cert "Your CA Name" --org com.yourcompany.vpn
```

---

# Examples

## Basic (uses .conf name for output and profile name)
./wg-conf-to-mobileconfig client.conf --cert "Acme Corp CA"

## Custom output filename + org
./wg-conf-to-mobileconfig home.conf home-vpn --cert "Jamf CA" --org com.home.secure

## Output

home-vpn.mobileconfig → signed, ready for upload
Profile appears in System Settings > Profiles as home

---

# Options

| Option              | Description                                                                 |
|---------------------|-----------------------------------------------------------------------------|
| `<config.conf>`     | Input WireGuard config file                                                 |
| `[output_name]`     | Optional output base name (default: config filename without `.conf`)        |
| `--cert "Name"`     | **Required** – Exact name of the signing certificate in the keychain        |
| `--org com.example` | Payload identifier prefix (default: `com.yourcompany.wireguard`)            |
| `--help`            | Show help message                                                           |

---

# Verify your signing certificate

```bash
security find-identity -v -p codesigning
```

Look for your cert name. Example:

1) ABC123... "Acme Corp CA" (CSSMERR_TP_NOT_TRUSTED)

---

# Install on Mac (locally)

```bash
sudo profiles install -path=client.mobileconfig
```
**Verify:**
```bash
scutil --nc list
```

---

# Deploy with Jamf Pro

1. Upload .mobileconfig to Computers > Configuration Profiles
2. Scope to devices
3. Users see profile under System Settings > Profiles

---

# License

MIT License – Free to use, modify, and distribute
