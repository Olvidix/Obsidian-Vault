# 1. Activar la aceleración de gráficos 3D y vm-tools
```
sudo apt update
sudo apt install open-vm-tools-desktop -y
sudo reboot
```
# 2.Super script
```
#!/bin/bash

# Script ultra agresivo de limpieza y optimización de Kali Linux
# Ejecutar como root o con sudo
set -e

echo "=== 1. Actualizando listas de paquetes y full-upgrade ==="
sudo apt update
sudo apt full-upgrade -y

echo "=== 2. Eliminando paquetes huérfanos y purgando configuraciones ==="
sudo apt autoremove --purge -y
sudo apt clean
sudo apt autoclean

echo "=== 3. Limpiando temporales del sistema y caches de usuarios ==="
sudo rm -rf /tmp/* /var/tmp/*
for user_cache in /home/*/.cache; do
    [ -d "$user_cache" ] && rm -rf "$user_cache"/*
done
echo "Temporal y caches de usuarios limpiados."

echo "=== 4. Limpieza de logs y backups antiguos ==="
sudo journalctl --vacuum-time=7d
sudo find /var/log -type f -name "*.log" -exec truncate -s 0 {} \;
sudo find /var/log -type f -name "*.gz" -delete
sudo find /var/backups -type f -name "*.tar*" -delete
echo "Logs y backups antiguos limpiados."

echo "=== 5. Reparando paquetes pendientes y dependencias ==="
sudo apt -f install -y
sudo dpkg --configure -a

echo "=== 6. Detectando y purgando kernels antiguos automáticamente ==="
OLD_KERNELS=$(dpkg --list | awk '/^rc/ && /linux-image-[0-9]/ {print $2}')
if [ ! -z "$OLD_KERNELS" ]; then
    echo "Kernels antiguos encontrados: $OLD_KERNELS"
    sudo apt purge -y $OLD_KERNELS
else
    echo "No hay kernels antiguos para purgar."
fi

echo "=== 7. Limpiando caches de navegadores pesados ==="
for BROWSER_CACHE in /home/*/.mozilla /home/*/.cache/chromium /home/*/.cache/google-chrome /home/*/.cache/brave; do
    if [ -d "$BROWSER_CACHE" ]; then
        rm -rf "$BROWSER_CACHE"/*
        echo "Cache borrado: $BROWSER_CACHE"
    fi
done

echo "=== 8. Limpiando descargas antiguas (>30 días) ==="
for DOWNLOAD_DIR in /home/*/Downloads; do
    if [ -d "$DOWNLOAD_DIR" ]; then
        find "$DOWNLOAD_DIR" -type f -mtime +30 -exec rm -f {} \;
        echo "Archivos antiguos eliminados: $DOWNLOAD_DIR"
    fi
done

echo "=== 9. Limpiando snaps, flatpaks y paquetes obsoletos ==="
if command -v snap &>/dev/null; then
    sudo snap remove $(snap list --all | awk '/disabled/{print $1}') || true
fi

if command -v flatpak &>/dev/null; then
    flatpak uninstall --unused -y || true
fi

sudo apt autoremove --purge -y
sudo apt clean

echo "=== 10. Optimización de caches de memoria y disco ==="
sudo sync
sudo sh -c 'echo 3 > /proc/sys/vm/drop_caches'
echo "Memoria optimizada."

echo "=== 11. Actualizando GRUB ==="
sudo update-grub

echo "=== 12. Estado final del sistema ==="
df -h
systemctl list-units --state=failed

echo "=== Limpieza y optimización ULTRA completada ✅ ==="
```