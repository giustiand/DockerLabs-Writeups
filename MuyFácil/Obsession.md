# Obsession
**Herramientas y recursos usados**
- nmap
- searchsploit
- hydra

# Enumeración

Empezamos con un escaneo de los puertos.

`sudo nmap -p- --open -sC -sS -sV --min-rate=5000 -n -Pn -vvv 172.17.0.2 -oN Obsession`

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
![O_1](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/Obsession/O_1.jpg)  

Tenemos 3 puertos abiertos, 21, 22 y 80.  
Podemos notar que el puerto 21 tiene habilitado un servicio FTP que permite el login como anonymous.  

# Explotación
Accedemos al servicio FTP proporcionando tanto como usuario que como contraseña "anonyomous".  

![O_2](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/Obsession/O_2.jpg)   

Una vez dentro podemos mirar el contenido y bajar los ficheros para ver si podemos sacar algo util.  

![O_3](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/Obsession/O_3.jpg)  

![O_4](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/Obsession/O_4.jpg)  

Aparentemente no encontramos nada util, a parte unos posibles nombres de usuarios.  
Intentamos a ver si con el fuzzing web sacamos algo más cierto.  

![O_5](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/Obsession/O_5.jpg)  

Vemos que hay dos rutas muy interesantes, "backup" e "important", vamos a ver el contenido.  

![O_6](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/Obsession/O_6.jpg)  

Esta información es muy valiosa ya que ahora sabemos que hay un usuario llamado "russoski", por lo tanto usaremos `hydra` para realizar un ataque de forza bruta al servicio SSH.  

![O_7](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/Obsession/O_7.jpg)  

Accedemos a SSH con la contraseña que hemos descubierto y lanzamos el comando `sudo -l` para ver di podemos escalar de privilegios facilmente. 

# Escalada de privilegios
![O_8](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/Obsession/O_8.jpg)  

Perfecto, ya estamos dentro i podemos mirar en GTFOBins si hay alguna forma que nos permita escalar los privilegios.

![O_9](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/Obsession/O_9.jpg)   

Encontramos este comando que podría funcionar, entonces lo ejecutamos

![O_10](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/Obsession/O_10.jpg)   

Y listo! Ya somos root!



