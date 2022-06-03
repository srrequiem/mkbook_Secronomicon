# TTY

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

# Host discovery

## Ping Sweep

### Windows

**Powershell**

`1..255 | % {echo "172.16.2.$_"; ping -n 1 -w 100 172.16.2.$_} | Select-String ttl`

### Linux

#### Bash one-liner

`subnet="10.10.110." && for ip in {0..254}; do ping -c 1 -t 1 $subnet$ip  > /dev/null && echo "[+] Host found $subnet$ip"; done`

#### Nmap

TODO: Poner comando

# Proxy debugging

TODO: Agregar comandos de shellhacks.com para redireccionar todo el tráfico de la consola a burpsuite

# Windows

## Crackmapexec `(Pwn3d!)`

`cmd.exe /c reg add HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System /v LocalAccountTokenFilterPolicy /t REG_DWORD /d 1 /f`