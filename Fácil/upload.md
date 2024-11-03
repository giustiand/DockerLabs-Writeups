# Upload

# Enumeración

Comenzamos con un escaneo de los puertos.  

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

Tenemos únicamente un puerto abierto, el 80.  
Echamos un vistazo para ver qué hay.    

![U](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/upload/U_2.jpg)     

Intentamos subir una reverse shell en PHP para ver si es posible.  

![U](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/upload/U_3.jpg)    

![U](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/upload/U_4.jpg)    

¡Perfecto!   
Ahora realizaremos un poco de fuzzing web en busca de una carpeta donde se pueda haber guardado este archivo.  
Para ello, ejecutamos el siguiente comando:  

`sudo gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt`  

![U](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/upload/U_5.jpg)   

¡Listo!  
Ahora revisamos la ruta http://172.17.0.2/uploads y nos ponemos a la escucha en nuestra máquina Kali.  
Una vez hecho esto, abrimos el archivo reverse_shell.php.  

![U](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/upload/U_6.jpg)     

Bien.  
Ahora podemos ejecutar el comando `sudo -l` para ver si podemos aprovechar algún archivo para escalar privilegios.  

![U](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/upload/U_7.jpg)   

Entonces, consultaremos la web GTFOBins.  

![U](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/upload/U_8.jpg)     

Ahora simplemente ejecutaremos el comando `sudo -u root /usr/bin/env /bin/bash`  

![U](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/upload/U_9.jpg)     

¡Y listo!   
Ahora somos root.  




