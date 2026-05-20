# LIGOLO-NG

Herramienta alternativa al uso de **chisel** y **socat**, al funcionar como una VPN, en lugar de usar **socks** es más ligera y eficiente. [Github](https://github.com/nicocha30/ligolo-ng).

## Instalacion

1. Debemos descargar tanto el `agente` como el `proxy`.
2. Creamos la interfaz de `ligolo` y la inicializamos:

```bash
sudo ip tuntap add user root mode tun ligolo
sudo ip link set ligolo up
```

## Configuracion

1. Nuestra máquina de atacante levantamos el **proxy**:

```bash
sudo ./proxy -selfcert 
```

> Podemos asignarle un puerto de escucha, si no lo hacemos por defecto utilizará **10601**.

2.  En la máquina **intermediaría 1** ejecutamos el agente:

```bash
sudo ./agent -connect 10.0.2.4:11601 -ignore-cert -retry
```

> Tras ejecutar esto veremos en el `proxy` que un `agente` se ha conectado, ahora vamos a configurarlo:

3. Configuración a través de la consola interactiva del **proxy** el primer salto:

```bash
ligolo-ng » session          # lista de sesiones
ligolo-ng » 1                # selecciona la sesión

~~sudo ip route add 10.0.3.0/24 dev ligolo~~

VM1 » start
```

Para el 2 hop

```bash
sudo ip tuntap add user root mode tun ligolo2
sudo ip link set ligolo2 up
```

```
VM1 » listener_add --addr 0.0.0.0:11601 --to 127.0.0.1:11601 --tcp
```

```bash
sudo ./agent -connect 10.0.3.4:11601 -ignore-cert -retry
```


```bash
ligolo-ng » session          
ligolo-ng » 2                

~~sudo ip route add 10.0.4.0/24 dev ligolo2~~

VM2 » start --tun ligolo2
```






Para encadenar puertos:
```
VM1 »listener_add --addr 0.0.0.0:4444 --to 10.0.2.4:4444 --tcp
VM2 »listener_add --addr 0.0.0.0:4444 --to 10.0.3.4:4444 --tcp
```

Asi encadenamos por ejemplo el 4444

Si queremos hacer un tercer hop a veces no ahce falta hacer la interfaz ligolo3






```BASH
VM2 » listener_add --addr 0.0.0.0:11601 --to 127.0.0.1:11601 --tcp
```

ligolo-ng » interface_add_route --name ligolo --route 10.0.3.0/24
ligolo-ng » listener_add --addr 0.0.0.0:8100 --to 127.0.0.1:8100
```


4.  En la máquina **intermediaría 2** ejecutamos de nuevo el agente:

```bash
./agent -connect 10.0.2.4:11601 -ignore-cert -retry
```

5. Configuración a través de la consola interactiva del **proxy** el segundo salto:

```bash
ligolo-ng » session          # lista de sesiones
ligolo-ng » 2                # selecciona la sesión
ligolo-ng » interface_add_route --name ligolo --route 20.0.0.0/24
ligolo-ng » listener_add --addr 0.0.0.0:8200 --to 127.0.0.1:8200
```