[[Pcredz]]

| Filtro Wireshark                                | Descripción                                                                                          |
| ----------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| ip.addr == 56.48.210.13                         | Filtra paquetes con una dirección IP específica.                                                     |
| tcp.port == 80                                  | Filtra paquetes por puerto (HTTP en este caso).                                                      |
| http                                            | Filtros para tráfico HTTP.                                                                           |
| dns                                             | Filtra el tráfico DNS, útil para supervisar la resolución de nombres de dominio.                     |
| tcp.flags.syn == 1 && tcp.flags.ack == 0        | Filtra paquetes SYN (utilizados en TCP), útiles para detectar intentos de escaneo o conexión.        |
| icmp                                            | Filtra paquetes ICMP (utilizados para ping), útil para reconocimiento o problemas de red.            |
| http.request.method == "POST"                   | Filtros para solicitudes HTTP POST, que pueden contener contraseñas u otra información confidencial. |
| tcp.stream eq 53                                | Filtra un flujo TCP específico, ayudando a rastrear una conversación entre dos hosts.                |
| eth.addr == 00:11:22:33:44:55                   | Filtra paquetes desde/hacia una dirección MAC específica.                                            |
| ip.src == 192.168.24.3 && ip.dst == 56.48.210.3 | Filtra el tráfico entre dos direcciones IP específicas, útil para rastrear comunicación entre hosts. |
