# Extract nmap information
extractPorts() {
    ports="$(grep -oP '\d{1,5}/open' "$1" | awk -F/ '{print $1}' | tr '\n' ',' | sed 's/,$//')"
    ip_address="$(grep -oP '\d{1,3}(\.\d{1,3}){3}' "$1" | sort -u)"
    
    echo -e "\n[*] Extracting information...\n" > extractPorts.tmp
    echo -e "\t[*] IP Address: $ip_address" >> extractPorts.tmp
    echo -e "\t[*] Open ports: $ports\n" >> extractPorts.tmp
    
    echo -n "$ports" | xclip -sel clip
    echo -e "[*] Ports copied to clipboard\n" >> extractPorts.tmp
    
    cat extractPorts.tmp
    rm extractPorts.tmp
}
