[[👀Password Attacks & Hunting]]
Una vez obtenido el SAM , SYTEM y SECURITY, procedemos a usar secretdumps.py:

```
impacket-secretsdump -sam sam.save -security security.save -system system.save
LOCAL
```

Nos lo dará con el formato:
```
Dumping local SAM hashes (uid:rid:lmhash:nthash)
```

Los antiguos son los lmhashes y los más actuales son los nthashes

Para crackearlos usaremos los nt, en hashcat es el -m 1000

## NTDS.dit
```
impacket-secretsdump -ntds NTDS.dit -system SYSTEM LOCAL
```
## Hashes DGC2
También veremos que el secretsdump.py también nos da hashes DGC2, estos son los hashes de los que hemos hablado que podían esta en la caché del SECURITY, para ello usaremos el -m 2100

![[Pasted image 20250920173444.png]]
 [[⛓️‍💥 Password Cracking]]