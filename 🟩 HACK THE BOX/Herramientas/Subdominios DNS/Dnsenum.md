Para listar subdominios.
``` Subdominios_dentro_de_subdominios
dnsenum --dnsserver [IP] --enum -p 0 -s 0 -o subdomains.txt -f /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt [NOMBRE DEL SUBDOMINIO]
```

```Subdominios
dnsenum --enum [IP/NOMBRE] -f /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -r
```

Ejemplo:
```1.
dnsenum --dnsserver 10.129.130.122 --enum -p 0 -s 0 -o subdomains.txt -f /opt/useful/seclists/Discovery/DNS/fierce-hostlist.txt dev.inlanefreight.htb
```

```2.
dnsenum --enum inlanefreight.com -f /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -r
```



[[Script para subdominios con DIG]]
[[DIG]]
[[🟩 HACK THE BOX/Herramientas/Web/GoBuster|GoBuster]]