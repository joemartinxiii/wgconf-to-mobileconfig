#!/bin/bash

# wg-conf-to-mobileconfig - Convert WireGuard .conf to signed mobileconfig
# Fully generalized: customer-agnostic, configurable signing & identifiers
# Usage: ./wg-conf-to-mobileconfig <wireguard.conf> [output_base] [--cert "Cert Name"] [--org com.example.vpn]

set -e

# === Defaults ===
CERT_NAME=""
ORG="com.yourcompany.wireguard"
OUTPUT_BASE=""
CONF_FILE=""

# === Help ===
show_help() {
    cat << 'EOF'
Usage: ./wg-conf-to-mobileconfig <wireguard.conf> [output_base] [options]

Positional:
  <wireguard.conf>           Input WireGuard config file
  [output_base]              Optional output filename base (default: conf name)

Options:
  --cert "Certificate Name"  Exact name of signing cert in keychain (required)
  --org  com.example.vpn     PayloadIdentifier prefix (default: com.yourcompany.wireguard)
  --help                     Show this help

Examples:
  ./wg-conf-to-mobileconfig client.conf --cert "Acme Corp CA"
  ./wg-conf-to-mobileconfig home.conf home-vpn --cert "Jamf CA" --org com.home.secure
EOF
    exit 1
}

# === Parse Arguments ===
while [[ $# -gt 0 ]]; do
    case $1 in
        --cert)
            CERT_NAME="$2"
            shift 2
            ;;
        --org)
            ORG="$2"
            shift 2
            ;;
        --help)
            show_help
            ;;
        *)
            if [ -z "$CONF_FILE" ]; then
                CONF_FILE="$1"
            elif [ -z "$OUTPUT_BASE" ]; then
                OUTPUT_BASE="$1"
            else
                echo "Error: Too many positional arguments."
                show_help
            fi
            shift
            ;;
    esac
done

# === Input Validation ===
if [ -z "$CONF_FILE" ]; then
    echo "Error: Missing .conf file."
    show_help
fi
if [ ! -f "$CONF_FILE" ]; then
    echo "Error: File '$CONF_FILE' not found."
    exit 1
fi
if [ -z "$CERT_NAME" ]; then
    echo "Error: --cert is required."
    show_help
fi

# === Derive Names ===
PROFILE_NAME="$(basename "${CONF_FILE%.conf}")"
OUTPUT_BASE="${OUTPUT_BASE:-$PROFILE_NAME}"
UNSIGNED="${OUTPUT_BASE}.unsigned.mobileconfig"
SIGNED="${OUTPUT_BASE}.mobileconfig"

# === Generate UUIDs ===
TOP_UUID=$(uuidgen | tr '[:lower:]' '[:upper:]')
PAYLOAD_UUID=$(uuidgen | tr '[:lower:]' '[:upper:]')
VPN_UUID=$(uuidgen | tr '[:lower:]' '[:upper:]')

TOP_ID_UUID=$(echo "$TOP_UUID" | tr -d '-')
VPN_ID_UUID=$(echo "$VPN_UUID" | tr -d '-')

# === Extract Endpoint ===
ENDPOINT=$(awk '
/^\[Peer\]$/ { in_peer=1; next }
/^\[/ { in_peer=0 }
in_peer && /^Endpoint[[:space:]]*=[[:space:]]*/ {
    sub(/^Endpoint[[:space:]]*=[[:space:]]*/, "");
    gsub(/[[:space:]]+$/, "");
    print; exit
}' "$CONF_FILE")

if [ -z "$ENDPOINT" ]; then
    echo "Error: No Endpoint found in [Peer] section."
    exit 1
fi

# === Read and Escape Config ===
CONF_CONTENT=$(cat "$CONF_FILE")
ESCAPED_CONF=$(printf '%s\n' "$CONF_CONTENT" | sed 's/&/\&amp;/g; s/</\&lt;/g; s/>/\&gt;/g')

# === Generate Plist ===
cat > "$UNSIGNED" << EOF
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>PayloadDisplayName</key>
	<string>WireGuard Configuration Profile</string>
	<key>PayloadType</key>
	<string>Configuration</string>
	<key>PayloadVersion</key>
	<integer>1</integer>
	<key>PayloadIdentifier</key>
	<string>${ORG}.${TOP_ID_UUID}</string>
	<key>PayloadUUID</key>
	<string>${TOP_UUID}</string>
	<key>PayloadContent</key>
	<array>
		<dict>
			<key>PayloadDisplayName</key>
			<string>VPN</string>
			<key>PayloadType</key>
			<string>com.apple.vpn.managed</string>
			<key>PayloadVersion</key>
			<integer>1</integer>
			<key>PayloadIdentifier</key>
			<string>com.wireguard.macos.${VPN_ID_UUID}</string>
			<key>PayloadUUID</key>
			<string>${PAYLOAD_UUID}</string>
			<key>UserDefinedName</key>
			<string>${PROFILE_NAME}</string>
			<key>VPNType</key>
			<string>VPN</string>
			<key>VPNSubType</key>
			<string>com.wireguard.macos</string>
			<key>VendorConfig</key>
			<dict>
				<key>WgQuickConfig</key>
				<string>
${ESCAPED_CONF}
				</string>
			</dict>
			<key>VPN</key>
			<dict>
				<key>RemoteAddress</key>
				<string>${ENDPOINT}</string>
				<key>AuthenticationMethod</key>
				<string>Password</string>
			</dict>
		</dict>
	</array>
</dict>
</plist>
EOF

# === Validate Plist ===
if ! plutil -lint "$UNSIGNED" > /dev/null 2>&1; then
    echo "Error: Generated plist is invalid:"
    cat "$UNSIGNED"
    rm -f "$UNSIGNED"
    exit 1
fi

# === Sign ===
echo "Signing with: '$CERT_NAME'"

if ! security cms -S -N "$CERT_NAME" -i "$UNSIGNED" -o "$SIGNED"; then
    echo "Error: Signing failed."
    echo "Available signing identities:"
    security find-identity -v -p codesigning
    rm -f "$UNSIGNED" "$SIGNED" 2>/dev/null || true
    exit 1
fi

# === Cleanup ===
rm -f "$UNSIGNED"

# === Success ===
echo "Success: $SIGNED created"
echo "   Profile name: '$PROFILE_NAME'"
echo "   Identifier:   ${ORG}.${TOP_ID_UUID}"
echo "   Signed with:  '$CERT_NAME'"
