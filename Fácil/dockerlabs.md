# Dockerlabs

# Enumeración

Empezamos con un escaneo de los puertos.

`sudo nmap -p- --open -sC -sS -sV --min-rate=5000 -n -Pn -vvv 172.17.0.2 -oN Dockerlabs`  

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

Tenemos solo un puerto abierto, el 80.  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/dockerlabs/D_1.jpg)    

Echamos un vistazo a la web.  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/dockerlabs/D_2.jpg)     

A primera vista, no hay nada interesante, tampoco en el código fuente.    
Entonces, realizamos un poco de fuzzing con la herramienta **gobuster**.    
Ejecutamos el comando:   

`sudo gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,txt,bak`  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/dockerlabs/D_3.jpg)   

Encontramos algunos recursos interesantes, sobre todo el **machine.php**, que nos permite cargar archivos.  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/dockerlabs/D_4.jpg)     

Por lo tanto, intentamos cargar una reverse shell en PHP.   

![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/dockerlabs/D_5.jpg)      

Bueno, por lo que vemos, al parecer solo podemos subir archivos con extensión .zip.    
Probamos a ver si podemos eludir esta limitación.    
Abrimos la herramienta **Burpsuite** y capturamos la solicitud.  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/dockerlabs/D_6.jpg)      

Ahora lo que haremos será pasar esta misma solicitud al "Intruder" para ver si podemos subir archivos con otras extensiones.    
Lo primero que debemos hacer es seleccionar la extensión que necesitamos controlar.  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/dockerlabs/D_7.jpg)    

Luego, iremos a la pestaña de **payload** y seleccionaremos un diccionario.  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/dockerlabs/D_8.jpg)   

Finalmente, haremos clic en "Start attack".  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/dockerlabs/D_9.jpg)   

Al observar los resultados, vemos que podemos cargar, además de los archivos .zip, también los archivos .phar.  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/dockerlabs/D_10.jpg)     

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/dockerlabs/D_11.jpg)   

Entonces, lo que haremos será cambiar la extensión de nuestro archivo de .php a .phar y probaremos a subirlo.  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/dockerlabs/D_12.jpg)     

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/dockerlabs/D_13.jpg)    

¡Perfecto!    
Ahora nos pondremos a la escucha en nuestra máquina Kali y abriremos el archivo para ver si nos devuelve una reverse shell.  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/dockerlabs/D_14.jpg)    

¡Estamos dentro!  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/dockerlabs/D_15.jpg)    

Si ejecutamos el comando `sudo -l`, vemos que podemos abusar de dos binarios: **cut** y **grep**.  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/dockerlabs/D_16.jpg)    

Lo que tenemos que hacer es encontrar un archivo para intentar escalar privilegios.    
Para ello, ejecutamos el comando:  

`find / -type f -name *.txt 2>/dev/null`  

Y vemos que en la carpeta **/opt** encontramos un archivo interesante.  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/dockerlabs/D_17.jpg)      

Lo abrimos:  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/dockerlabs/D_18.jpg)      

Ahora consultamos la web de GTFObins y abusaremos del binario **cut** para escalarnos a root.  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/dockerlabs/D_19.jpg)      

Ejecutaremos el comando:  

`sudo /usr/bin/cut -d "" -f1 "/root/clave.txt"` y listo, ya tenemos la clave de root.  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/dockerlabs/D_20.jpg)      

Ahora simplemente ejecutaremos el comando `su` y le proporcionaremos esta contraseña... ¡y listo!    
¡Ya somos root!  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/dockerlabs/D_21.jpg)      







