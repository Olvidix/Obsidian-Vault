# Enumeración de usuarios
Con una Wordlist de posibles nombres generada por ejemplo con [[Username Anarchy]] podremos usarlo:

```
./kerbrute userenum --dc [IP_DC] --domain [DOMINIO.LOCAL] names.txt -o Usuarios_Validos.txt
```

Con esto nos dará usuarios validos para ese entorno de AD


Si la autenticación por kerberos está descativada nos dará el bloque de cifrado de la pre-autenticacion es decir su hash, el cual podremos crackear con `hascat -m 18200`

Con esto y la contraseña que podremos sacar con [[Netexec]] podremos conectarnos mediante [[WinRM (5985,5986)]]

# PasswordSpraying
Con la wordlist generada anteriormente podremos hacer passwordspraying:
```
./kerbrute passwordspray --dc [IP_DC] --domain [DOMINIO.LOCAL] Usuarios_Validos.txt [CONTRASEÑA]
```