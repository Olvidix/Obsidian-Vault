Principalmente para acceder a recursos compartidos
# Comandos
### -L
Para listar recursos

### -N
Para conectarse con una Null sessions (Sin credenciales)


## Uso

Conectarse a SMB con una Null session:
```
smbclient -L //10.129.14.128 -N
```


Conectarse a SMB con credenciales
```
smbclient //[IP]/[RECURSO] -U [USUARIO]%[CONTRASEÑA]
```

Conectarse a un recurso del SMB:
```
smbclientt //[IP]/[RECURSO]
```

## Comandos dentro

### ![COMANDO]
SMB permite ejecutar comandos del sistema local si le ponemos el signo "!" delante del comando. Por ejemplo: ls, cat, etc...
PERO SOLO DEL SISTEMA LOCAL , ES DECIR DEL NUESTRO!
### get
Para descargar archivos
### put
Para subir archivos




[[RPCclient]]