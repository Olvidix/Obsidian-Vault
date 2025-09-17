# Onesixtyone

Para sacar por fuerza bruta los las cadenas de comunidad:
```
sudo apt install onesixtyone

onesixtyone -c /usr/share/wordlists/seclists/Discovery/SNMP/snmp.txt [IP]
```
(Puede llevar bastante tiempo)(Se podria llegar a requerir de hacer un diccionario personalizado con una herramienta como crunch)

# SNMPwalk

Consultar y recorrer información SNMP :
```
snmpwalk -v2c -c [CADENA_DE_COMUNIDAD] [IP]
```
Una cadena de comunidad típica es "public"
# Braa

Consultar OIDs específicos rápidamente:
```
braa [CADENA_DE_COMUNIDAD]@[IP]:.1.3.6.*
```


Una cadena de comunidad típica es "public"