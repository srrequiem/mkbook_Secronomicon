# SSH + Proxychains

**Redirección local de puertos**

```bash
# Sintaxis
ssh -N -L bind_address:port:host:host_port [username@address]

# Setup
sudo ssh -N -L 0.0.0.0:445:192.168.1.110:445 student@10.11.0.128

# Uso
smbclient -L 127.0.0.1 -U Administrator
```

TODO: Añadir imagen de flujo de data en puertos y evidencia

* * *

**Redirección remota de puertos**

```bash
# Sintaxis
ssh -N -R bind_address:port:host:host_port [username@address]

# Setup
sudo ssh -N -R 10.11.0.4:2221:127.0.0.1:3306 kali@10.11.0.4

# Uso
TODO
```

TODO: Añadir imagen de flujo de data en puertos y evidencia

* * *

**Redirección dinámica de puertos**

```bash
# Sintaxis
ssh -N -D address_to_bind_to:port_to_bind_to [username@server_address]

# Setup
sudo ssh -N -D 127.0.0.1:8080 student@10.11.0.128
## Edición de Proxychains /etc/proxychains.conf
# [ProxyList]
#
#
# socks4  127.0.0.1 8080

# Uso
proxychains nmap --top-ports=20 -sT -Pn 192.168.1.110
```

TODO: Añadir imagen de flujo de data en puertos y evidencia

* * *

**Parámetros SSH**

- `-N` No levantará prompt de comandos.
- `-L` Indica redirección local.
- `-R` Indica redirección remota.
- `-D` Indica redirección dinámica.

# Chisel

Ref: https://0xdf.gitlab.io/2020/08/10/tunneling-with-chisel-and-ssf-update.html

## Server

`chisel server -p 9001 --reverse`

## Client

`chisel client ip_server:server_port R:socks`