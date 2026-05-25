---
tags:
  - active-directory
  - kerberos
  - winrm
  - pentesting
  - lateral-movement
---

## La idea central en una frase

Cuando te conectas a una máquina con Kerberos (vía WinRM), recibes un ticket válido **solo para esa máquina**. Si desde ahí quieres saltar a una tercera máquina, no tienes credenciales que reenviar, y te deniegan el acceso.

---

## ¿Por qué pasa esto? La diferencia clave entre NTLM y Kerberos

| Autenticación | ¿Qué se guarda en memoria? | ¿Sirve para saltar? |
|---|---|---|
| **Contraseña/NTLM** (ej: PSExec) | El hash NTLM queda en LSASS | Sí — la máquina puede reusarlo para autenticarte en otro recurso |
| **Kerberos** (ej: WinRM) | Solo un **TGS** específico para *este* servicio | No — sin TGT no puedes pedir tickets nuevos |

La clave: **el TGT (Ticket Granting Ticket) NUNCA se reenvía a la sesión remota**. Solo viaja el TGS, que es un "pase" válido únicamente para el servicio al que te conectaste (ej: `HTTP/DEV01`).

---

## El escenario típico

```
[Tu Kali] ──WinRM──> [DEV01] ──LDAP──> [DC01]
   Salto 1: OK         Salto 2: DENEGADO
```

1. Te conectas a `DEV01` con `evil-winrm` usando credenciales de `backupadm`.
2. En `DEV01`, intentas correr `PowerView` → `Get-DomainUser` (que consulta al DC).
3. **Error**: `DEV01` necesita un TGT de `backupadm` para pedir un TGS hacia `DC01`, pero ese TGT nunca llegó.

Por eso si corres `mimikatz` en `DEV01`, verás que la sesión de `backupadm` está **vacía** (sin NTLM, sin contraseña, sin TGT). Solo existe el proceso `wsmprovhost.exe` corriendo en su contexto, pero sin material criptográfico para autenticarse hacia adelante.

Si ejecutas `klist` ves solo **un ticket**: el TGS para `HTTP/DEV01` (que es el servicio de WinRM). Nada más.

### Comprobación rápida con mimikatz

```powershell
Enter-PSSession -ComputerName DEV01 -Credential INLANEFREIGHT\backupadm
cd 'C:\Users\Public\'
.\mimikatz "privilege::debug" "sekurlsa::logonpasswords" exit
```

En la salida verás que para `backupadm` los campos `wdigest`, `kerberos` y `credman` están vacíos (`Password : (null)`). No hay material para reautenticarse.

### Comprobación con klist (desde la sesión WinRM)

```
Cached Tickets: (1)

#0> Client: backupadm @ INLANEFREIGHT.LOCAL
    Server: HTTP/ACADEMY-AEN-DEV01.INLANEFREIGHT.LOCAL @ INLANEFREIGHT.LOCAL
    ...
```

Solo el TGS de HTTP. No hay `krbtgt/...` (TGT).

---

## La excepción: Unconstrained Delegation

Si `DEV01` tiene **delegación sin restricciones**, el cliente envía su TGT junto con el TGS. Entonces `DEV01` cachea tu TGT y puede pedir tickets a nombre tuyo hacia cualquier otro servicio. Aquí no hay problema de doble salto (y de hecho es una vulnerabilidad clásica que se explota).

---

## Soluciones alternativas

### Solución 1: `PSCredential` explícito (funciona desde `evil-winrm`)

Le pasas las credenciales **con cada comando** mediante el flag `-Credential`:

```powershell
# 1. Construir el objeto de credenciales
$SecPassword = ConvertTo-SecureString '!qazXSW@' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\backupadm', $SecPassword)

# 2. Usar -Credential en cada cmdlet que toque el DC
Get-DomainUser -SPN -Credential $Cred | select samaccountname   # ← funciona
Get-DomainUser -SPN                                              # ← falla (doble salto)
```

**Cómo funciona**: el comando lleva las credenciales en claro como argumento, así que el segundo host puede autenticarse por sí mismo contra el DC.

**Limitación**: muchas herramientas no aceptan `-Credential`, así que no siempre es viable.

---

### Solución 2: `Register-PSSessionConfiguration` (requiere GUI/PowerShell elevado)

Registras una configuración de sesión en el host destino que corre como ese usuario:

```powershell
# En el host de ataque Windows, en PowerShell elevado:
Register-PSSessionConfiguration -Name backupadmsess -RunAsCredential inlanefreight\backupadm

# Reiniciar el servicio WinRM
Restart-Service WinRM

# Conectarse usando la nueva configuración
Enter-PSSession -ComputerName DEV01 -Credential INLANEFREIGHT\backupadm -ConfigurationName backupadmsess
```

Ahora `klist` en la sesión muestra un **TGT real** (`krbtgt/INLANEFREIGHT.LOCAL`), no solo el TGS de HTTP. Puedes consultar el DC directamente sin pasar `-Credential` en cada cmdlet:

```powershell
[DEV01]: PS C:\Users\Public> get-domainuser -spn | select samaccountname
# ← funciona sin pasar credenciales
```

**Limitaciones importantes**:
- No funciona desde `evil-winrm` (no hay popup para credenciales).
- Necesitas PowerShell **elevado**.
- No funciona desde PowerShell en Linux (Parrot/Ubuntu) por cómo gestiona Kerberos.
- Solo sirve si trabajas desde un host Windows con GUI, o desde un host comprometido al que entras por RDP.

---

## Otras alternativas (no desarrolladas)

- **CredSSP** — habilita explícitamente el reenvío de credenciales (peligroso, deja la contraseña en memoria del segundo host).
- **Port forwarding** — tunelizas el tráfico para que el "segundo salto" sea en realidad un primer salto desde tu perspectiva.
- **Process injection / sacrificial process** — te inyectas en un proceso que ya corre con un TGT válido del usuario objetivo.

---

## Resumen mental

> **El doble salto de Kerberos = "el TGT se queda contigo, no viaja". Sin TGT en el host intermedio, no se pueden generar tickets nuevos hacia el siguiente recurso.**

Las soluciones se reducen a tres ideas:
1. **Llevar las credenciales contigo** en cada comando (`PSCredential`).
2. **Hacer que el segundo host se autentique por sí mismo** como el usuario (`Register-PSSessionConfiguration`, CredSSP).
3. **Aprovechar configuraciones inseguras** (unconstrained delegation).

---

## Comandos clave de referencia rápida

| Comando | Para qué |
|---|---|
| `klist` | Ver tickets Kerberos en caché |
| `mimikatz "sekurlsa::logonpasswords"` | Ver credenciales en memoria (LSASS) |
| `Enter-PSSession -ComputerName X -Credential Y` | Sesión WinRM interactiva |
| `Register-PSSessionConfiguration -Name X -RunAsCredential Y` | Crear endpoint WinRM que corre como otro usuario |
| `Restart-Service WinRM` | Aplicar cambios de configuración WinRM |
| `tasklist /V | findstr <user>` | Ver procesos corriendo como un usuario |

---

## Enlaces relacionados (Obsidian) (Habra que enlazarlo a los buenos!!)

- Kerberos - Conceptos básicos
- Unconstrained Delegation
- Constrained Delegation
- PowerView - Cheatsheet
- evil-winrm - Cheatsheet
- Mimikatz - Cheatsheet
- Lateral Movement en AD