Uso básico
```
smbmap -H [IP]
```

```
smbmap -u [USER] -p [PASS] -d [DOMINIO.LOCAL] -H [IP]
```

Para ver un recurso en concreto:
```
smbmap -H [IP] -r [RECURSO]
```

Para descargar:
```
smbmap -H [IP] --download "[RECURSO]\[NOMBRE_ARCHIVO]"
```

Para subir:
```
smbmap -H [IP] --upload [ARCHIVO] "[RECURSO]\[ARCHIVO]"
```


[[RPCclient]]