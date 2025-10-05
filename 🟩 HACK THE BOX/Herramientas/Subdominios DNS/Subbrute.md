Primero añadimos la IP al /etc/hosts
```
10.129.203.6 inlanefreight.htb
```

Segundo añadimos la IP al archivo resolvers
```
echo "[IP_OBJETIVO]" > resolvers.txt
```

Después ejecutamos la herramienta:
```
python3 subbrute.py -p [DOMINIO] -s names.txt -r resolvers.txt
```
Ejemplo:
```
python3 subbrute.py -p inlanefreight.htb -s names.txt -r resolvers.txt
```

