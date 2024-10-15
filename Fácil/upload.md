# Upload

# Enumeración

Empezamos con un escaneo de los puertos.

`sudo nmap -p- --open -sC -sS -sV --min-rate=5000 -n -Pn -vvv 172.17.0.2 -oN Upload`  

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

![U](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/upload/U_1.jpg)   

Tenemos solamente un puerto abierto, el 80.  
Echamos un ojo para ver que hay.  

![U](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/upload/U_2.jpg)     

Probamos a subir una reverse shell en php para ver si nos deja.  

![U](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/upload/U_3.jpg)    

![U](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/upload/U_4.jpg)    

Perfecto! 
Ahora tendremos que hacer un poco de fuzzing web a la busqueda de una carpeta donde se pueda haber guardado este fichero.  
Para ello ejecutamos el comando:  

`sudo gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt`

![U](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/upload/U_5.jpg)   

Listo! 
Ahora miramos la ruta http://172.17.0.2/uploads y nos ponemos a la escucha en nuestra máquina kali y una vez hecho abrimos el fichero reverse_shell.php.  

![U](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/upload/U_6.jpg)     

Bien! 
Ahora podemos dar el comando `sudo -l` para ver si podemos aproecharnos de algún fichero para escalar de privilegios.  

![U](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/upload/U_7.jpg)   

Miraremos entonces la web GTFOBins.  

![U](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/upload/U_8.jpg)     

Ahora simplemente ejecutaremos el comando `sudo -u root /usr/bin/env /bin/bash`

![U](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/upload/U_9.jpg)     

Y listo!  
Ya somos root!  




