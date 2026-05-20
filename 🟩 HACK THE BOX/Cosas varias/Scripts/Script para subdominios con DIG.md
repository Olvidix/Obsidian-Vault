```
for sub in $(cat /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt);do dig $sub.[DOMINIO] @[IP] | grep -v ';\|SOA' | sed -r '/^\s*$/d' | grep $sub | tee -a subdomains.txt;done
```

Ejemplo:
```
for sub in $(cat /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt);do dig $sub.inlanefreight.htb @10.129.203.6 | grep -v ';\|SOA' | sed -r '/^\s*$/d' | grep $sub | tee -a subdomains.txt;done
```


[[Dnsenum]]
[[DIG]]
[[Apuntes Desordenados/🟩 HACK THE BOX/Herramientas/Web/GoBuster|GoBuster]]