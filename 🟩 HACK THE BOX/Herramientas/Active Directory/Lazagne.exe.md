Primero nos transferimos el ejecutable a nuestra victima Windows, y después de esto la ejecutaremos con el siguiente comando:
```cmd
.\LaZagne.exe all
```

Con esto usaremos todos los módulos de lazagne pero si queremos alguno solo en concreto:

| **Module** | **Description**                                                                                   |
| ---------- | ------------------------------------------------------------------------------------------------- |
| browsers   | Extracts passwords from various browsers including Chromium, Firefox, Microsoft Edge, and Opera   |
| chats      | Extracts passwords from various chat applications including Skype                                 |
| mails      | Searches through mailboxes for passwords including Outlook and Thunderbird                        |
| memory     | Dumps passwords from memory, targeting KeePass and LSASS                                          |
| sysadmin   | Extracts passwords from the configuration files of various sysadmin tools like OpenVPN and WinSCP |
| windows    | Extracts Windows-specific credentials targeting LSA secrets, Credential Manager, and more         |
| wifi       | Dumps WiFi credentials                                                                            |
[[SessionGopher.ps1]]                               [[EvilTree]]