# Extract nmap information

```
sudo nano ~/.zshrc
```

```
# Extract nmap information
extractPorts() {
    [ -z "$1" ] && { echo "Usage: extractPorts <file>"; return 1; }
    [ ! -f "$1" ] && { echo "File not found: $1"; return 1; }

    ports="$(grep -oP '\d{1,5}/open' "$1" | awk -F/ '{print $1}' | tr '\n' ',' | sed 's/,$//')"
    ip_address="$(grep -oP '\d{1,3}(\.\d{1,3}){3}' "$1" | sort -u)"

    echo -e "\n[*] Extracting information...\n" > /tmp/extractPorts.tmp
    echo -e "\t[*] IP Address: $ip_address" >> /tmp/extractPorts.tmp
    echo -e "\t[*] Open ports: $ports\n" >> /tmp/extractPorts.tmp
    echo -n "$ports" | xclip -sel clip 2>/dev/null
    echo -e "[*] Ports copied to clipboard\n" >> /tmp/extractPorts.tmp

    cat /tmp/extractPorts.tmp
    rm -f /tmp/extractPorts.tmp
}

alias ep='extractPorts'
```

```
source ~/.zshrc
```