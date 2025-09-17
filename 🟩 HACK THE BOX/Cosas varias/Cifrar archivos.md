En las pruebas de penetración vamos a encontrarnos con archivos y para moverlos de una manera segura deberemos cifrar los archivos antes si es que el método de transferencia no se ha podido hacer mediante: SSH, SFTP y HTTPS, que ya de por si son seguros.

## Linux
#### Cifrar
```
openssl enc -aes256 -iter 100000 -pbkdf2 -in [Archivo] -out [NOMBRE].enc
```
Y pones una CONTRASEÑA.
#### Descifrar
```
openssl enc -d -aes256 -iter 100000 -pbkdf2 -in [NOMBRE].enc -out [ARCHIVO]
```
Y después pones la CONTRASEÑA.

## Windows

#### Importamos el modulo
```PowerShell
Import-Module .\Invoke-AESEncryption.ps1
```
#### Cifrar
```powerShell
Invoke-AESEncryption -Mode Encrypt -Key "[CONTRASEÑA]" -Path .\[ARCHIVO]
```
Genera un archivo con el mismo nombre pero con extensión `.aes`
#### Descifrar
```PowerShell
Invoke-AESEncryption -Mode Decrypt -Key "[CONTRASEÑA]" -Path .\[ARCHIVO].aes
```
