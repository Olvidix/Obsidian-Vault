Primero vamos a hacer una prueba sin AWS simplemente nuestro PC y la raspberry, vamos a necesitar:

- La raspberry y todas sus movidas
- Wireguard tanto en las Raspberry como en nuestra Kali que va a hacer de servidor

## Rasberry Pi Imager en nuestro PC y flasheamos la SD

  

Para ello vamos a la web oficial de [Raspberry Pi software – Raspberry Pi](https://www.raspberrypi.com/software/)

Y lo configuramos:

1. Seleccionamos nuestra raspberry

2. Seleccionamos nuestro sistema operativo (Elegir SO > Other EspecificPurpose OS > Kali Linux > Raspberry 3, 4, 400 ,5 and 500 (64-bit))

3. Seleccionamos nuestra SD/Micro SD


Una vez clickeado en siguiente nos mostrara la siguiente pantalla:

Le damos a Editar ajustes, y en el apartado General:
 
Aquí lo configuramos a nuestro gusto, pero siempre activando y rellenando esto porque despues lo necesitaremos más adelante. También podemos añadir la Wifi directamente para más comodidad, pero en principio usaremos el cable de red que es más estable

Después en servicios:

Esto lo ponemos asi de momento para conectarnos luego por SSH y que sea más cómodo de instalar las cosas. El apartado de Opciones lo podemos dejar por defecto

## Instalamos Kali

Metemos la SD ya flasheada en la rasberry y tardara un rato la primera vez, pero después ya se ira iniciando más rápido, las credenciales son las que hayamos puesto en la configuración del Imager.

Una vez dentro actualizamos rapberry (También lo haremos con el servidor Kali):

```Shell
sudo apt update && sudo apt upgrade -y
```

Va bastante lento la raspberry paciencia (A mí se me ha quedado a medias como 4 veces hasta que he podido actualizarla entera)

## Ahora vamos a instalar Wireguard en ambos dispositivos

En la raspberry:

  ```Shell
sudo apt install -y wireguard iptables-persistent resolvconf
sudo mkdir -p /etc/wireguard
```
 
En el servidor con Kali:

```Shell
sudo apt install -y wireguard iptables-persistent
sudo mkdir -p /etc/wireguard
sudo sysctl -w net.ipv4.ip_forward=1
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
```

## Generamos Claves de WireGuard  

Importante nunca compartir la clave privada, solo la pública

En la Raspberry Pi:

```Shell
cd /etc/wireguard #Tendremos que ponernoes en root
sudo wg genkey | sudo tee client_private.key | wg pubkey | sudo tee client_public.key
sudo chmod 600 client_private.key
sudo chmod 644 client_public.key

#Ver la clave pública para añadir al servidor
sudo cat client_public.key
```

En el servidor Kali:

```Shell
cd /etc/wireguard
sudo wg genkey | sudo tee server_private.key | wg pubkey | sudo tee server_public.key
sudo chmod 600 server_private.key
sudo chmod 644 server_public.key

# Ver la clave pública para copiarla a la Raspberry
sudo cat server_public.key
```

## Configuraciones

### Configuración del servidor Kali

Creamos el archivo de configuración:

  ```Shell
sudo nano /etc/wireguard/wg0.conf
```

Y le metemos lo siguiente:

  ```Shell
[Interface]
PrivateKey = [[SERVER_PRIVATE_KEY]]   # contenido de server_private.key
Address = 10.200.0.1/24
ListenPort = 51820

PostUp = iptables -A FORWARD -i wg0 -o eth0 -j ACCEPT; iptables -A FORWARD -i eth0 -o wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -o eth0 -j ACCEPT; iptables -D FORWARD -i eth0 -o wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

# Peer Raspberry Pi
[Peer]
PublicKey = [[CLIENT_PUBLIC_KEY]]     # la public key de la Pi
AllowedIPs = 10.200.0.2/32, 172.16.0.0/24
```

  ⚠️ IMPORTANTE

  `eth0` debe ser la interfaz del bastion que da salida a Internet (si es ens3, eno1, cambia nombre).

  ⚠️ 172.16.0.0/24 → sustituir por la LAN privada del cliente.

 Levantamos la interfaz y le habilitamos el arranque automático:

  ```Shell
sudo systemctl enable wg-quick@wg0
sudo systemctl start wg-quick@wg0
sudo wg show
```

### Configuración de la Raspberry Pi

  Creamos el archivo de configuración:

  ```Shell
sudo nano /etc/wireguard/wg0.conf
```

 Y le metemos lo siguiente:

  ```Shell
[Interface]
PrivateKey = [[CLIENT_PRIVATE_KEY]]   # contenido de client_private.key
Address = 10.200.0.2/32
DNS = 1.1.1.1  

# Si la Raspberry usa WiFi, interfaz correcta = wlan0
# Si usa cable, será eth0
PostUp   = iptables -A FORWARD -i wg0 -o wlan0 -j ACCEPT; iptables -A FORWARD -i wlan0 -o wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -o wlan0 -j ACCEPT; iptables -D FORWARD -i wlan0 -o wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o wlan0 -j MASQUERADE

[Peer]
PublicKey = [[SERVER_PUBLIC_KEY]]     # contenido de server_public.key
Endpoint = [[IP_KALI(o Publica del bastion)]]:51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25

```

 ESTA ULTIMA RED TENDREMOS QUE PONER LA QUE NOS DE EL CLIENTE LA DE EJEMPLO ESTA`172.16.0.0/24` (En ese formato de, se puede añadir todos los rangos que queramos)

  ⚠️ Aquí está una de las correcciones críticas:

  `AllowedIPs = 0.0.0.0/0`

  → Necesario para que el bastion pueda redirigir tráfico hacia la LAN del cliente.
  
Levantamos la interfaz y le habilitamos el arranque automático:

  ```Shell
sudo systemctl enable wg-quick@wg0
sudo systemctl start wg-quick@wg0
sudo wg show
```

## Activamos Forwarding en la Raspberry

  ```Shell
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## Guardamos las reglas iptables en la Raspberry

```Shell
sudo iptables-save | sudo tee /etc/iptables/rules.v4
```

## Verificamos la conexión

### Desde el servidor Kali:

```Shell
ping 10.200.0.2
```

### Desde la Raspberry Pi:

```Shell
ping 10.200.0.1
```

## Verificamos el pivoting a la LAN

### Desde el servidor Kali:


```Shell
ping 172.16.0.1
```


O desde el PC Windows:

```Shell
ssh kali@BASTION_IP
ping 172.16.0.1
```
# Esquema resumen

```Shell
[ PC Windows ]
        ↓ SSH
[ Bastion (Kali/Ubuntu) ]
        ↓ WireGuard (10.200.0.0/24)
[ Raspberry en cliente ]
        ↓ Pivot / NAT
[ LAN Cliente ]
```