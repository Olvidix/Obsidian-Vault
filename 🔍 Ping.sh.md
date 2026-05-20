```
#!/bin/bash

# Script de ping sweep paralelo para redes pivotadas con Ligolo
# Pregunta al usuario el rango de red y muestra hosts vivos en pantalla

read -p "Introduce el rango de red (ej: 10.0.3.0/24): " NETWORK

MAX_JOBS=20

echo "Escaneando la red $NETWORK ..."

# Obtener el primer y último host del rango
IFS='/' read -r NET PREFIX <<< "$NETWORK"
# Convertimos el rango a 1..254 (simplificado para /24)
for i in $(seq 1 254); do
    IP="${NET%.*}.$i"
    (
        ping -c 1 -W 1 $IP &> /dev/null && echo "Host alive: $IP"
    ) &

    # Limitar procesos paralelos
    while [ $(jobs | wc -l) -ge $MAX_JOBS ]; do
        sleep 0.1
    done
done

wait

echo "Escaneo completado."
```


Si esta quitado el ping a veces falla pero tenemos la versión mejorada:

```
#!/bin/bash

read -p "Rango (ej: 172.24.0.0/24): " NETWORK
MAX_JOBS=30

echo "Escaneando $NETWORK ..."

NET="${NETWORK%.*}.0"

for i in $(seq 1 254); do
    IP="${NETWORK%.*}.$i"

    (
        # ICMP
        ping -c 1 -W 1 $IP &>/dev/null && {
            echo "Host alive (ICMP): $IP"
            exit
        }

        # TCP (válido para contenedores sin ICMP)
        for p in 22 80 443 8080 8000; do
            timeout 0.3 bash -c "echo >/dev/tcp/$IP/$p" 2>/dev/null && {
                echo "Host alive (TCP:$p): $IP"
                exit
            }
        done
    ) &

    while [ $(jobs | wc -l) -ge $MAX_JOBS ]; do
        sleep 0.1
    done
done

wait
echo "Done."
```