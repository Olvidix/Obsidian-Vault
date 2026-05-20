```PowerShell
PS C:\WINDOWS\system32> Get-AppxPackage -AllUsers Microsoft.YourPhone | Remove-AppxPackage                              

PS C:\WINDOWS\system32> reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced" /v Start_AccountNotifications /t REG_DWORD /d 0 /f
```

