# John The Reaper
#### Para sacar el hash de un archivo
```
[Extension_Archivo]2john [Archivo] > hash
```
#### Identificar el Hash
```
hashid -j [HASH]
```
También puedes ver la lista [aquí](https://pentestmonkey.net/cheat-sheet/john-the-ripper-hash-formats)
### Wordlist Mode
```
john --wordlist=<wordlist_file> <hash_file>
```

O podemos usar --format si sabemos el formato exacto:
```
john --format=[FORMATO] [HASH]
```
### Single Crack Mode
Por ejemplo con credenciales Linux:
```passwd
r0lf:$6$ues25dIanlctrWxg$nZHVz2z4kCy1760Ee28M1xtHdGoy0C2cYzZ8l2sVa1kIa8K9gAcdBP.GI6ng/qA4oaMrgElZ1Cb9OeXO4Fvy3/:0:0:Rolf Sebastian:/home/r0lf:/bin/bash
```
Con este formato se puede usar john para hacer un ataque personalizado al usuario:
```
john --single passwd
```
### Incremental Mode
```
john --incremental <hash_file>
```

Se puede customizar aquí:
```
grep '# Incremental modes' -A 100 /etc/john/john.conf
```

# HashCat
#### Identificar el Hash
```
hashid -m [HASH]
```
También puedes ver una lista extensa [aquí](https://hashcat.net/wiki/doku.php?id=example_hashes)

Con estos pasos tendremos el modo (-m) con el que funcionara hashcat
### Wordlist Mode
```
hashcat -a 0 -m 0 [HASH] [WORDLIST]
```

Para que sea más efectivo podemos añadir alguna regla con -r:
```
hashcat -a 0 -m 0 [HASH] [WORDLIST] -r [PATH_TO_RULE]
```

Para ver las reglas disponibles:
```
ls -l /usr/share/hashcat/rules
```

### Mask Mode
```
hashcat -a 3 -m 0 [HASH] '[MASCARA]'
```

| Símbolo | Set de Caracteres                   |
| ------- | ----------------------------------- |
| ?l      | abcdefghijklmnopqrstuvwxyz          |
| ?u      | ABCDEFGHIJKLMNOPQRSTUVWXYZ          |
| ?d      | 0123456789                          |
| ?h      | 0123456789abcdef                    |
| ?H      | 0123456789ABCDEF                    |
| ?s      | «space»!"#$%&'()*+,-./:;<=>?@[]^_`{ |
| ?a      | ?l?u?d?s                            |
| ?b      | 0x00 - 0xff                         |
Símbolos para crear nuestra máscara (Ejemplo: ?u?l?l?l?l?d?s)



## Para crear wordlists que combinen palabras dentro de la wordlist que hemos creado:

Primero generamos una wordlist.txt con información del usuario y después:

```
hashcat --stdout wordlist.txt wordlist.txt -a 1 > combos.txt
```

## Mutar diccionarios con hashcat
Siguiendo esto:

|**Función**|**Descripción**|
|---|---|
|`:`|No hacer nada|
|`l`|Todas las letras en minúsculas|
|`u`|Todas las letras en mayúsculas|
|`c`|Escriba con mayúscula la primera letra y con minúscula las demás.|
|`sXY`|Reemplazar todas las instancias de X con Y|
|`$!`|Añade el carácter de exclamación al final.|

Podemos hacer rules tal como este ejemplo

| custom.rule  | Output (ej: password) |
| ------------ | --------------------- |
| :            | password              |
| c            | Password              |
| so0          | passw0rd              |
| c so0        | Passw0rd              |
| sa@          | p@ssword              |
| c sa@        | P@ssword<br>          |
| c sa@ so0    | P@ssw0rd              |
| $!           | password!             |
| $! c         | Password!             |
| $! so0       | passw0rd!             |
| $! sa@       | p@ssword!             |
| $! c so0     | Passw0rd!             |
| $! c sa@     | P@ssword!             |
| $! so0 sa@   | p@ssw0rd!             |
| $! c so0 sa@ | P@ssw0rd!             |

# Casos concretos
## Descifrar .gzip con OpenSSL
```
for i in $(cat rockyou.txt);do openssl enc -aes-256-cbc -d -in [Archivo.gzip] -k $i 2>/dev/null| tar xz;done
```

## Descifrar BitLocker (.vhd)
```
bitlocker2john -i Backup.vhd > backup.hashes
grep "bitlocker\$0" backup.hashes > backup.hash
cat backup.hash


hashcat -a 0 -m 22100 backup.hash  [WORDLIST]

```
### Para montar unidades de bitlocker
#### Windows
Hacemos doble click en el archivo y ponemos la pass obtenida

#### Linux
```
sudo apt-get install dislocker

sudo mkdir -p /media/bitlocker
sudo mkdir -p /media/bitlockermount

sudo losetup -f -P Backup.vhd
sudo dislocker /dev/loop1p1 -u[PASSWORD] -- /media/bitlocker
sudo mount -o loop /media/bitlocker/dislocker-file /media/bitlockermount


cd /media/bitlockermount/
ls -la
```

Y para desmontarla
```
sudo umount /media/bitlockermount
sudo umount /media/bitlocker
```