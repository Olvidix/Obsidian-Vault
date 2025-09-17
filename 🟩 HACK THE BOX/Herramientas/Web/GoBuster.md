## -T
para los hilos
## -K
Para ignorar los SSL/TLS
## -o
Para guardar la salida



Ejemplo normal fuzzing de directorios
```
gobuster dir -u http://inlanefreight.htb:52584/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
```



Ejemplo de fuzzing en subdirectorios:
También se puede usar para el descubrimiento de virtualhosts:

```
gobuster vhost -u http://[IP] -w [WORDLIST] --append-domain
```

```EJEMPLO
gobuster vhost -u http://inlanefreight.htb:37954 -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt --append-domain
```

Si vamos a usar un nombre DNS primero tenemos que configurar el /etc/hosts para que resuelva a la IP que nos ha dado y el puerto si es que nos lo dan después en el comando

[[Script para subdominios con DIG]]
[[DIG]]
[[Dnsenum]]