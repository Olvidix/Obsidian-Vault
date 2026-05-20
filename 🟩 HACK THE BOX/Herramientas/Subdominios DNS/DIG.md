
| Dominio                         | Descripción                                                                                                                                                                                                                    |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `dig domain.com`                | Realiza una búsqueda de registro A predeterminada para el dominio.                                                                                                                                                             |
| `dig domain.com A`              | Recupera la dirección IPv4 (registro A) asociada con el dominio.                                                                                                                                                               |
| `dig domain.com AAAA`           | Recupera la dirección IPv6 (registro AAAA) asociada con el dominio.                                                                                                                                                            |
| `dig domain.com MX`             | Encuentra los servidores de correo (registros MX) responsables del dominio.                                                                                                                                                    |
| `dig domain.com NS`             | Identifica los servidores de nombres autorizados para el dominio.                                                                                                                                                              |
| `dig domain.com TXT`            | Recupera cualquier registro TXT asociado con el dominio.                                                                                                                                                                       |
| `dig domain.com CNAME`          | Recupera el registro de nombre canónico (CNAME) del dominio.                                                                                                                                                                   |
| `dig domain.com SOA`            | Recupera el registro de inicio de autoridad (SOA) para el dominio.                                                                                                                                                             |
| `dig @1.1.1.1 domain.com`       | Especifica un servidor de nombres específico para consultar; en este caso 1.1.1.1                                                                                                                                              |
| `dig +trace domain.com`         | Muestra la ruta completa de resolución DNS.                                                                                                                                                                                    |
| `dig -x 192.168.1.1`            | Realiza una búsqueda inversa en la dirección IP 192.168.1.1 para encontrar el nombre de host asociado. Es posible que deba especificar un servidor de nombres.                                                                 |
| `dig +short domain.com`         | Proporciona una respuesta breve y concisa a la consulta.                                                                                                                                                                       |
| `dig +noall +answer domain.com` | Muestra solo la sección de respuesta de la salida de la consulta.                                                                                                                                                              |
| `dig domain.com ANY`            | Recupera todos los registros DNS disponibles para el dominio (Nota: muchos servidores DNS ignoran `ANY`las consultas para reducir la carga y evitar abusos, según [RFC 8482](https://datatracker.ietf.org/doc/html/rfc8482) ). |

```
dig axfr @10.129.215.201 inlanefreight.htb
```

Este comando indica que `dig`se debe solicitar una transferencia de zona completa ( `axfr`) al servidor DNS responsable de `zonetransfer.me`. Si el servidor está mal configurado y permite la transferencia, recibirá una lista completa de registros DNS del dominio, incluidos todos los subdominios.


[[Script para subdominios con DIG]]
[[Dnsenum]]
[[Apuntes Desordenados/🟩 HACK THE BOX/Herramientas/Web/GoBuster|GoBuster]]