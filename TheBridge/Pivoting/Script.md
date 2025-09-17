```
#!/bin/bash  
  
# Red que se quiere escanear  
RED="10.0.30"  
  
# Rango de hosts donde podría estar Pivoting VM 2  
INICIO=3  
FIN=20  
  
# Puertos abiertos que se buscan  
PUERTOS=(22 80)  
  
# En este archivo se escriben las IPs encontradas  
SALIDA="ips_abiertas.txt"  
> "$SALIDA"  # Vacía el archivo antes de empezar  
  
echo "[*] Escaneando IPs con puertos abiertos..."  
echo "[!] Pulsa 'q' en cualquier momento para detener la ejecución."  
  
for host in $(seq $INICIO $FIN); do  
  IP="$RED.$host"  
  
  # Verifica si el usuario ha pulsado 'q'  
  read -t 1 -n 1 key  
  if [[ $key = "q" ]]; then  
    echo "[!] Escaneo cancelado por el usuario."  
    break  
  fi  
  
  for port in "${PUERTOS[@]}"; do  
    if proxychains timeout 3 bash -c "echo > /dev/tcp/$IP/$port" 2>/dev/null; then  
      echo "$IP tiene el puerto $port abierto"  
      echo "$IP:$port" >> "$SALIDA"  
    else  
      echo "$IP puerto $port cerrado o sin respuesta"  
    fi  
  done  
done  
  
echo "[+] IPs con puertos abiertos guardadas en: $SALIDA"
```