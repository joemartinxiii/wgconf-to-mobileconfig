# `wg-conf-to-mobileconfig`

**Convert a WireGuard `.conf` file into a signed macOS `.mobileconfig` profile** — perfect for deploying via **Jamf Pro**, **Intune**, or manual installation.

---

## Features

- Converts any standard WireGuard `.conf` to a valid `com.apple.vpn.managed` profile  
- Automatically escapes XML special characters  
- Generates unique UUIDs and identifiers per profile  
- Signs with **any certificate** in your macOS keychain  
- Fully **customer-agnostic** — no hard-coded org or cert names  
- Works with **Jamf Pro**, **Apple Configurator**, or `profiles` CLI  

---

## Requirements

- **macOS** (tested on macOS 14 Sonoma & 15 Sequoia)
- `uuidgen`, `plutil`, `security` (built-in)
- **Signing certificate** with private key in **System** or **login** keychain
- WireGuard app installed on target Macs (for `com.wireguard.macos`)

---

## Usage

```bash
./wg-conf-to-mobileconfig <config.conf> [output_name] --cert "Your CA Name" --org com.yourcompany.vpn
