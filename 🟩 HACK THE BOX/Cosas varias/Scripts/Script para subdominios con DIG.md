```
for sub in $(cat /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt);do dig $sub.inlanefreight.htb @[IP] | grep -v ';\|SOA' | sed -r '/^\s*$/d' | grep $sub | tee -a subdomains.txt;done
```



[[Dnsenum]]
[[DIG]]
[[🟩 HACK THE BOX/Herramientas/Web/GoBuster|GoBuster]]