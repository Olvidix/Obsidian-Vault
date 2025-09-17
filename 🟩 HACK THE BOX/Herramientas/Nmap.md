Principalmente para escanear puertos abiertos y sus correspondientes servicios y versiones

Para sacar una lista con solo las IPs activas:
```
sudo nmap -sn 10.0.2.0/24 -oA [lista] | grep for | cut -d" " -f5
```

## -sT
La flag "-sT" sirve para cercionarnos si un estado del puerto es correcto ya que hace la conexión completa el "Three-Way-Handshake" y verifica el estado del puerto
Se usa cuando la precisión prevale sobre la sigilosidad.

## -sS
Por otro lado la flag "-sS" no establece la conexión completa solo el primer paso y es muchísimo mas sigilosa, pero no siempre es preciso al 100%

## -sU
La flag "-sU" sirve para los puertos UDP pero no se suelen dar mucho pero nunca esta de mas probarlos. Este escaneo es muchísimo mas lento en comparación a los otros

## -sV
La flag "-sV" sirve para detectar versiones de los servicios que corren por los puertos

## -sC
Escaneo de scripts. "--script " para especifacar categoria de script o nombres en concreto de estos

## -sA
Te dice solo si un perto esta no esta filtered 

## -D RND:5 
Método de escaneo de señuelos ( `-D`) es la opción correcta. Con este método, Nmap genera varias direcciones IP aleatorias insertadas en la cabecera IP para ocultar el origen del paquete enviado. Con este método, podemos generar aleatoriamente ( `RND`) un número específico (por ejemplo: `5`) de direcciones IP separadas por dos puntos ( `:`). Nuestra dirección IP real se coloca aleatoriamente entre las direcciones IP generadas.

## -S
Escanea el objetivo utilizando diferentes direcciones IP de origen

## -O
Realiza una detección de sistema operativo

## -A
Realiza detección de servicios, detección de SO, traceroute y utiliza scripts predeterminados para escanear el objetivo.

## --script-trace
Nmap también permite rastrear el progreso de los scripts de NSE a nivel de red si usamos esta opción en nuestros análisis.

## --source-port 53
Hace que **todos los paquetes de Nmap salgan con el puerto de origen `53`**, que es el puerto estándar de **DNS**.

## --stats-every=5s
Muestra el progreso del escaneo cada 5 segundos.

## --disable-arp-ping
Sirve para **desactivar el envío de paquetes ARP** cuando haces un escaneo en una red local.


- `-T 0` / `-T paranoid`
- `-T 1` / `-T sneaky`
- `-T 2` / `-T polite`
- `-T 3` / `-T normal`
- `-T 4` / `-T aggressive`
- `-T 5` / `-T insane`


# Scripts según servicios

```
Ruta por defecto          ---------->         /usr/share/nmap/scripts 
```

```
Para un servicio en concreto -------> ls /usr/share/nmap/scripts | grep [Servicio]
```

```
nmap --script [Serivio]* 10.129.14.128 -sV -p [Puerto/s del servicio]
```
# Escaneo super sigiloso contra firewalls
Super lento (Incluso horas) pero super sigiloso:
```
nmap -sS -n -Pn -D RND:5 -T 0  [IP] --stats-every=5s
```





Retos HTB evasion de Firewalls:
1:
```
nmap -sSV -A -O -n -Pn --top-ports 100 -T 3 10.129.9.93
```

2:
```
nmap -sV -v -p 53 -n -Pn -T 2 10.129.2.48 -D RND:5 --stats-every=5s
```

3:Hay que venir del puerto 53 del DNS para cumplirlo 
```
nmap -sV -p- -v -n -Pn -T 3 10.129.9.45 -D RND:5 --stats-every=5s --disable-arp-ping --source-port 53
```
Despues:
```
nc -nv -p 53 10.129.9.45 50000
```