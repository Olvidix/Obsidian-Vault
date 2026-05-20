Principalmente para enumerar informacion

Para conectarse anónimamente:
```
rpcclient -U "" -N [IP]
```

Para conectarse con credenciales:
```
rpcclient -U [USUARIO]%[CONTRASEÑA] [IP]
```

Para conectarse con un hash NTLM:
```
rpcclient -U [USUARIO] --pw-nt-hash [HASH_NTML] [IP]
```
## Comandos una vez dentro:

| `srvinfo`                 | Información del servidor.                                                             |
| ------------------------- | ------------------------------------------------------------------------------------- |
| `enumdomains`             | Enumerar todos los dominios que están implementados en la red.                        |
| `querydominfo`            | Proporciona información de dominio, servidor y usuario de los dominios implementados. |
| `netshareenumall`         | Enumera todas las acciones disponibles.                                               |
| `netsharegetinfo <share>` | Proporciona información sobre una acción específica.                                  |
| `enumdomusers`            | Enumera todos los usuarios del dominio.                                               |
| `queryuser <RID>`         | Proporciona información sobre un usuario específico.                                  |
| getdompwinfo              | Política de contraseñas                                                               |
| enumdomgroups             | Para enumerar grupos                                                                  |
| `querygroup <RID>`        | Para enumerar los usuarios dentro de un grupo especifico.                             |


Script automatizado para sacar los RID a través de fuerza bruta:
```
for i in $(seq 500 1100);do rpcclient -N -U "" [IP] -c "queryuser 0x$(printf '%x\n' $i)" | grep "User Name\|user_rid\|group_rid" && echo "";done
```
 
 
 
[[Samrdump.py]]
[[Enum4linux]]
[[Netexec]]
[[Smbmap]]
[[Apuntes Desordenados/🟩 HACK THE BOX/Herramientas/SMBclient|SMBclient]]






