## used
### 実行
- `chmod +x dns-check.sh`
### 通常のDNS設定を使うなら、
- `./dns-check.sh domains.txt`
### DNSサーバーを指定するなら、
- `./dns-check.sh domains.txt 8.8.8.8`
### CSVにも保存するなら、
- `./dns-check.sh domains.txt 8.8.8.8 result.csv`


## script
```bash
#!/usr/bin/env bash

set -u

# ============================================================
# DNS Record Checker
#
# Usage:
#   ./dns-check.sh domains.txt
#   ./dns-check.sh domains.txt 8.8.8.8
#   ./dns-check.sh domains.txt 8.8.8.8 result.csv
#
# Input:
#   example.com
#   www.example.com
#   api.example.net
#
# Output:
#   FQDN | CNAME | A | A_TTL | AAAA | AAAA_TTL | DNS Server | Status
#
# Multiple records are comma-separated.
# ============================================================

if ! command -v dig >/dev/null 2>&1; then
    echo "[ERROR] dig command not found."
    echo "Install it with:"
    echo "  sudo apt install dnsutils"
    exit 1
fi

if [[ $# -lt 1 ]]; then
    echo "Usage: $0 <fqdn_list> [dns_server] [output.csv]"
    exit 1
fi

INPUT_FILE="$1"
DNS_SERVER="${2:-}"
OUTPUT_FILE="${3:-}"

if [[ ! -f "$INPUT_FILE" ]]; then
    echo "[ERROR] File not found: $INPUT_FILE"
    exit 1
fi

# ------------------------------------------------------------
# DNS server option
# ------------------------------------------------------------

if [[ -n "$DNS_SERVER" ]]; then
    DNS_OPTION="@${DNS_SERVER}"
else
    DNS_OPTION=""
fi

# ------------------------------------------------------------
# CSV header
# ------------------------------------------------------------

if [[ -n "$OUTPUT_FILE" ]]; then
    echo '"FQDN","CNAME","A","A_TTL","AAAA","AAAA_TTL","DNS Server","Status"' \
        > "$OUTPUT_FILE"
fi

# ------------------------------------------------------------
# Header
# ------------------------------------------------------------

printf "%-30s | %-30s | %-35s | %-12s | %-45s | %-12s | %-20s | %s\n" \
    "FQDN" \
    "CNAME" \
    "A" \
    "A_TTL" \
    "AAAA" \
    "AAAA_TTL" \
    "DNS Server" \
    "Status"

printf '%*s\n' 205 '' | tr ' ' '-'

# ------------------------------------------------------------
# Process each FQDN
# ------------------------------------------------------------

while IFS= read -r FQDN || [[ -n "$FQDN" ]]; do

    # Remove CRLF
    FQDN="${FQDN//$'\r'/}"

    # Remove leading/trailing whitespace
    FQDN="$(echo "$FQDN" | xargs)"

    # Skip empty lines
    [[ -z "$FQDN" ]] && continue

    # Skip comments
    [[ "$FQDN" =~ ^# ]] && continue

    # Remove trailing dot
    FQDN="${FQDN%.}"

    # --------------------------------------------------------
    # DNS status
    # --------------------------------------------------------

    DNS_HEADER=$(dig $DNS_OPTION "$FQDN" A \
        +time=3 \
        +tries=1 \
        2>/dev/null)

    STATUS=$(echo "$DNS_HEADER" |
        awk '/status:/ {
            split($6,a,",")
            print a[1]
        }')

    [[ -z "$STATUS" ]] && STATUS="TIMEOUT"

    # --------------------------------------------------------
    # DNS server used
    # --------------------------------------------------------

    DNS_SERVER_USED=$(echo "$DNS_HEADER" |
        awk '/SERVER:/ {
            gsub(/[()]/, "", $3)
            print $3
        }')

    [[ -z "$DNS_SERVER_USED" ]] && DNS_SERVER_USED="-"

    # --------------------------------------------------------
    # CNAME
    # --------------------------------------------------------

    CNAME=$(dig $DNS_OPTION "$FQDN" CNAME \
        +noall \
        +answer \
        +time=3 \
        +tries=1 \
        2>/dev/null |
        awk '$4 == "CNAME" {
            print $5
        }' |
        sed 's/\.$//' |
        paste -sd ',' -)

    [[ -z "$CNAME" ]] && CNAME="-"

    # --------------------------------------------------------
    # A records
    #
    # Format:
    #   IP|TTL
    #
    # Example:
    #   1.2.3.4|300
    #   5.6.7.8|300
    # --------------------------------------------------------

    A_DATA=$(dig $DNS_OPTION "$FQDN" A \
        +noall \
        +answer \
        +time=3 \
        +tries=1 \
        2>/dev/null |
        awk '$4 == "A" {
            print $5 "|" $2
        }')

    A_RECORDS=$(echo "$A_DATA" |
        awk -F'|' 'NF {print $1}' |
        paste -sd ',' -)

    A_TTLS=$(echo "$A_DATA" |
        awk -F'|' 'NF {print $2}' |
        paste -sd ',' -)

    [[ -z "$A_RECORDS" ]] && A_RECORDS="-"
    [[ -z "$A_TTLS" ]] && A_TTLS="-"

    # --------------------------------------------------------
    # AAAA records
    #
    # Format:
    #   IPv6|TTL
    #
    # Example:
    #   2001:db8::1|60
    #   2001:db8::2|60
    # --------------------------------------------------------

    AAAA_DATA=$(dig $DNS_OPTION "$FQDN" AAAA \
        +noall \
        +answer \
        +time=3 \
        +tries=1 \
        2>/dev/null |
        awk '$4 == "AAAA" {
            print $5 "|" $2
        }')

    AAAA_RECORDS=$(echo "$AAAA_DATA" |
        awk -F'|' 'NF {print $1}' |
        paste -sd ',' -)

    AAAA_TTLS=$(echo "$AAAA_DATA" |
        awk -F'|' 'NF {print $2}' |
        paste -sd ',' -)

    [[ -z "$AAAA_RECORDS" ]] && AAAA_RECORDS="-"
    [[ -z "$AAAA_TTLS" ]] && AAAA_TTLS="-"

    # --------------------------------------------------------
    # Display
    # --------------------------------------------------------

    printf "%-30s | %-30s | %-35s | %-12s | %-45s | %-12s | %-20s | %s\n" \
        "$FQDN" \
        "$CNAME" \
        "$A_RECORDS" \
        "$A_TTLS" \
        "$AAAA_RECORDS" \
        "$AAAA_TTLS" \
        "$DNS_SERVER_USED" \
        "$STATUS"

    # --------------------------------------------------------
    # CSV
    # --------------------------------------------------------

    if [[ -n "$OUTPUT_FILE" ]]; then

        # Escape double quotes for CSV
        FQDN_CSV="${FQDN//\"/\"\"}"
        CNAME_CSV="${CNAME//\"/\"\"}"
        A_CSV="${A_RECORDS//\"/\"\"}"
        A_TTL_CSV="${A_TTLS//\"/\"\"}"
        AAAA_CSV="${AAAA_RECORDS//\"/\"\"}"
        AAAA_TTL_CSV="${AAAA_TTLS//\"/\"\"}"
        DNS_SERVER_CSV="${DNS_SERVER_USED//\"/\"\"}"
        STATUS_CSV="${STATUS//\"/\"\"}"

        printf '"%s","%s","%s","%s","%s","%s","%s","%s"\n' \
            "$FQDN_CSV" \
            "$CNAME_CSV" \
            "$A_CSV" \
            "$A_TTL_CSV" \
            "$AAAA_CSV" \
            "$AAAA_TTL_CSV" \
            "$DNS_SERVER_CSV" \
            "$STATUS_CSV" \
            >> "$OUTPUT_FILE"
    fi

done < "$INPUT_FILE"

# ------------------------------------------------------------
# Finished
# ------------------------------------------------------------

if [[ -n "$OUTPUT_FILE" ]]; then
    echo
    echo "[+] CSV saved to: $OUTPUT_FILE"
fi
```
