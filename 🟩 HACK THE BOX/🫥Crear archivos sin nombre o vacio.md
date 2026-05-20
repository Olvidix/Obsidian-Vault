cat > [NOMBRE] << EOF
#!/bin/bash
/bin/bash
EOF

Esto creara un archivo con eso dentro



para nombre pero contenido vacio:

touch -- '[NOMBRE]'

con el touch normal funcionaria pero si queremos un nombre que por ejemplo empiece por -- estaremos jodidos y así se solventa



# Y esto para uno ya existente por si no hay vim ni nano:
```
cat <<EOF > ARCHIVO.SH
#!/bin/bash
/usr/bin/sudo /usr/bin/chown -R sysadm:sysadm /root
EOF
```