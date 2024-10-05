# Pntopntobarra

# Enumeración

Empezamos con un escaneo de los puertos.

`sudo nmap -p- --open -sC -sS -sV --min-rate=5000 -n -Pn -vvv 172.17.0.2 -oN Pntopntobarra`  

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

Tenemos solamente dos puertos abiertos, el 22 y el 80.  

![Pn](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pntopntobarra/Pn_1.jpg)    

Miramos que hay en el puerto 80.  

![Pn](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pntopntobarra/Pn_2.jpg)    

Analizando el código fuente vemos esta línea que nos podríamos utilizar para probar a ver si estamos en frente a un caso de directory listing.  

![Pn](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pntopntobarra/Pn_3.jpg)    

Por lo tanto probamos a insertar la siguiente línea "../../../../etc/passwd".  

![Pn](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pntopntobarra/Pn_4.jpg)    

Efectivamente podemos sacar información valiosa.  
Ahora que sabemos que hay un usuario nombrado **nico** con la misma metodología podemos ver si hay una llave .ssh que nos permita de hacer login sin contraseña.  
Por lo tanto insertamos esa línea "../../../../home/nico/.ssh/id_rsa" y podemos sacar la llave.  

![Pn](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pntopntobarra/Pn_5.jpg)  

Ahora la copiaremos en nuestra maquina kali y le asignaremos los permisos 600 a través del comando `sudo chmod 600`. 
Hecho esto intentamos conectarnos por ssh con el comando:  

`ssh -i key nico@172.17.0.2`  

![Pn](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pntopntobarra/Pn_6.jpg)    

Una vez dentro ejecutamos el comando `sudo -l` para intentar escalar privilegios.  

![Pn](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pntopntobarra/Pn_7.jpg)    

Miramos en GTFOBins como podemos aprovecharnos de este binario para escalar a root.  

![Pn](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pntopntobarra/Pn_8.jpg)      

Ejecutamos el comando `sudo env /bin/sh` y listo!  
Ya somos root!  

![Pn](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pntopntobarra/Pn_9.jpg) 





