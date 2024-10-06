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

Tenemos solamente un puerto abierto, el 80.  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/dockerlabs/D_1.jpg)    

Echamos un ojo a la web.  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/dockerlabs/D_2.jpg)     

A primera vista no hay nada de interesante, tampoco en el codigo fuente.  
Entonces hacemos un poco de fuzzin con la herramienta **gobuster**.  
Ejecutamos el comando:  

`sudo gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,txt,bak`  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/dockerlabs/D_3.jpg)   

Encontramos unos cuantos recursos interesantes, sobretodo el **machine.php** que nos permite cargar ficheros.  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/dockerlabs/D_4.jpg)     

Probamos por lo tanto a cargar una reverse shell php.  

![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/dockerlabs/D_5.jpg)      

Bueno, por lo que vemos, al parecer, podemos subir solamente ficheros con extensión .zip.  
Probamos a ver si podemos bypassar esta limitación.  
Abrimos la herramienta **Burpsuite** y capturamos la petición.  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/dockerlabs/D_6.jpg)      

Ahora lo que haremos será pasar esta misma petición al "Intruder" para ver si podemos subir ficheros con otras extensiones.  
Por primera cosa tenemos que seleccionar la extensión que tenemos que controlar. 

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/dockerlabs/D_7.jpg)    

Luego le daremos a la pestaña payload y seleccionaremos un diccionario.  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/dockerlabs/D_8.jpg)   

Finalmente le daremos a "Start attack".  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/dockerlabs/D_9.jpg)   

Mirando los resultados vemos que podemos cargar, a parte de los ficheros .zip también los ficheros.phar.  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/dockerlabs/D_10.jpg)     

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/dockerlabs/D_11.jpg)   

Entonces lo que haremos será modificar la extensión de nuestro fichero de .php a .phar y probaremos a subirlo.  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/dockerlabs/D_12.jpg)     

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/dockerlabs/D_13.jpg)    

Perfecto! 
Ahora nos pondremos a la escucha en nuestra maquina kali y abriremos el fichero a ver si nos devuelve una reverse shell.  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/dockerlabs/D_14.jpg)    

Estamos dentro!  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/dockerlabs/D_15.jpg)    

Si le damos al comando `sudo -l` vemos que podemos abusar de dos binarios, cut y grep. 

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/dockerlabs/D_16.jpg)    

Lo que tenemos que hacer es encontrar un fichero para poder intentar escalar privilegios.  
Para ello le damos al comando:  

`find / -type f -name *.txt 2>/dev/null`  

y vemos que en la carpeta /opt encontramos un fichero interesante.  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/dockerlabs/D_17.jpg)      

Lo abrimos:  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/dockerlabs/D_18.jpg)      

Ahora miramos en la web de GTFObins y abusaremos del binario cut para escalarnos a root.  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/dockerlabs/D_19.jpg)      

Ejecutaremos el comando:  

`sudo /usr/bin/cut -d "" -f1 "/root/clave.txt"` y listo, ya tenemos la clave de root.  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/dockerlabs/D_20.jpg)      

Ahora simplemente le daremos al comando `su` y le proporcionaremos esta contraseña...y listo!  
Ya somos root!  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/dockerlabs/D_21.jpg)      







