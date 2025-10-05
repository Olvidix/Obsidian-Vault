
FUZZ [Donde queramos que se meta la wordlist]

-w [Para la wordlist]

-u [Para la URL]

-recursion [Para la recursividad]

-recursion-depth 1 [Profundidad de la recursividad]

-e .php [Extensiones]

-t 100 [Hilos]

-ic [Hace que no salgan los # molestos]

-v [Vervose]

# Para enumerar subdominios
```
ffuf -u http://FUZZ.inlanefreight.com/ -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt
```

# Para enumerar VHOSTS
```
ffuf -u http://inlanefreight.com/ -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -H 'Host: FUZZ.academy.htb'
```

# Ejemplo POST
```
ffuf -u http://admin.academy.htb:42333/admin/admin.php -w /usr/share/wordlists/SecLists/Discovery/Web-Content/burp-parameter-names.txt -X POST -d 'FUZZ=key' -H 'Content-Type: application/x-www-form-urlencoded'
```

# Ejemplo Get parameter
```
ffuf -u http://admin.academy.htb:42333/admin/admin.php?FUZZ=key -w /usr/share/wordlists/SecLists/Discovery/Web-Content/burp-parameter-names.txt
```

# Ejemplo habiendo descubierto parametro y valor
```
curl http://admin.academy.htb:42333/admin/admin.php -X POST -d 'id=73' -H 'Content-Type: application/x-www-form-urlencoded'
```

# Wordlists importantes:
## Fuzzing normal
```
/usr/share/wordlists/dirb/common.txt

/usr/share/wordlists/SecLists/Discovery/Web-Content/common.txt

/usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt
```

## DNS/VHOSTS
```
/usr/wordlist/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

## ? Parámetros
```
/usr/wordlist/seclists/Discovery/Web-Content/burp-parameter-names.txt
```

## Ids
```
for i in $(seq 1 1000); do echo $i >> ids.txt; done 
```

## Usernames
```
/usr/share/wordlists/SecLists/Usernames/xato-net-10-million-usernames.txt
```

## Passwords / Usernames
```
/usr/share/wordlists/rockyou.txt
```