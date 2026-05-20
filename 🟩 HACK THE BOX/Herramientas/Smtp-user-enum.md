Para enumerar nombres de usuario en el servicio SMTP [[3. SMTP (25) y SMTP cifrado (465-587)]]

Uso rápido:
```
smtp-user-enum -U /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt -t [IP]
```

Con dominio y especificando el modo:
```
smtp-user-enum -M RCPT -U /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt -D [DOMINIO.ALGO] -t [IP]
```

Como pudimos ver en [[3. SMTP (25) y SMTP cifrado (465-587)]] hay varias maneras de forma manual de sacar usuarios y con esta herramienta podemos usar esas formas especificando el `-M`: RCPT, EXPN o VRFY
