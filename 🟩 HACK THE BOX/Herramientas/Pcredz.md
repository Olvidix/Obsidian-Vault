[[Wireshark]]

Instalación:
```
sudo apt update

sudo apt install python3-pip libpcap-dev file

pip3 install Cython python-libpcap
```

Uso:
```
# extract credentials from a pcap file
python3 ./Pcredz -f file-to-parse.pcap -t -v

# extract credentials from all pcap files in a folder
python3 ./Pcredz -d /tmp/pcap-directory-to-parse/

# extract credentials from a live packet capture on a network interface (need root privileges)
python3 ./Pcredz -i eth0 -v
```

