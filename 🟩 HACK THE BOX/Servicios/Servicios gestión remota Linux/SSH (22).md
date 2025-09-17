Esto hace que fuerces la autenticación basada en contraseña:
```
ssh [USUARIO]@[IP] -o PreferredAuthentications=password
```


Para autenticarse con contraseña:
```
ssh [USUARIO]@[IP]
```

Para autenticarse con clave privada;
```
ssh -i id_rsa [USUARIO]@[IP]
```
