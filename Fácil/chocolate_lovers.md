# Chocolate lovers

# Enumeración

Empezamos con un escaneo de los puertos.  

`sudo nmap -p- --open -sC -sS -sV --min-rate=5000 -n -Pn -vvv 172.17.0.2 -oN Chocolate_lovers`

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

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/chocolate_lovers/C_1.jpg)   

Solo tenemos un puerto abierto: el 80.   
Echamos un vistazo para ver qué hay.   
Observamos que hay la página por defecto de Apache.  

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/chocolate_lovers/C_2.jpg)     

Revisamos si hay algo interesante en el código fuente.  

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/chocolate_lovers/C_3.jpg)     

Abrimos el recurso **/nibbleblog** para ver qué hay.  

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/chocolate_lovers/C_4.jpg)      

Ahora haremos un poco de fuzzing web.    
Ejecutamos el siguiente comando:    

`sudo gobuster dir -u http://172.17.0.2/nibbleblog -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php`  

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/chocolate_lovers/C_5.jpg)      

Hay una ruta, **admin.php**, que debería llamarnos la atención.    

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/chocolate_lovers/C_6.jpg)       

Como podemos ver, nos encontramos frente a un panel de inicio de sesión.    
Probamos con **admin/admin** y, por suerte, ya estamos dentro.  

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/chocolate_lovers/C_7.jpg)       

Al mirar en la página, vemos que es la versión 4.0.3 "Coffee".  

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/chocolate_lovers/C_8.jpg)      

Lo que podemos hacer es buscar en Google si encontramos algún exploit.   
Podemos ver que esta versión es vulnerable a la carga arbitraria de archivos.   
Lo que tenemos que hacer es activar el plugin llamado **My Image** y cargar nuestra reverse shell en PHP, sin prestar atención a los errores que aparezcan.  

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/chocolate_lovers/C_9.jpg)        

Ahora lo que tenemos que hacer es ir al recurso **http://172.17.0.2/nibbleblog/content/private/plugins/my_image/** y, una vez que nos hemos puesto a la escucha en nuestra máquina Kali, abrir el archivo llamado **image.php**.    

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/chocolate_lovers/C_10.jpg)        

¡Perfecto!  
Ya estamos dentro.  
Ahora lo que haremos será realizar el tratamiento de la shell para poder trabajar de forma más cómoda.  
Para ello, ejecutaremos los comandos:  

`script /dev/null -c bash` 

Después `Ctrl +Z` (el processo va en background)  

luego `stty raw -echo; fg` 

`reset xterm`

`export TERM=xterm`

`export SHELL=bash`  

Bien, ahora que podemos trabajar con más comodidad, lo que haremos será intentar escalar privilegios.    
Para ello, ejecutamos el comando:   

`sudo -l`  

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/chocolate_lovers/C_11.jpg)     

Ok, vemos que podemos aprovechar el binario de PHP para escalar a chocolate.    
Entonces, consultaremos GTFOBins para ver cómo hacerlo.    

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/chocolate_lovers/C_12.jpg)       

Ejecutamos entonces los comandos sugeridos y podemos convertirnos en chocolate.  

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/chocolate_lovers/C_13.jpg)         

Ahora probaremos a escalar a root.   
Ejecutamos el comando `sudo -l` y `find / -perm -4000 2>/dev/null`, pero no encontramos nada interesante.  

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/chocolate_lovers/C_14.jpg)   

Lo que haremos será subir a la máquina víctima el **pspy64** para ver si hay algo interesante.  
Por lo tanto, montaremos un servidor temporal en nuestra máquina Kali con el comando:   

`sudo python3 -m http.server`

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/chocolate_lovers/C_15.jpg)     

Ahora entraremos en la carpeta **/tmp** de la máquina víctima y descargaremos el archivo con la herramienta **wget**.    

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/chocolate_lovers/C_16.jpg)       

Ahora le daremos permisos de ejecución con el comando `chmod +x pspy64` y lo ejecutaremos.  

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/chocolate_lovers/C_17.jpg)    

Como podemos observar, en la carpeta **/opt** hay un script en PHP llamado **script.php** que se ejecuta cada 5 segundos.   
Dado que no tenemos acceso a nano y no podemos editar este archivo, lo que haremos será eliminarlo y subir a la máquina víctima un archivo llamado **script.php** que contendrá nuestra reverse shell.    
Una vez que hayamos editado el archivo, montaremos de nuevo un servidor en nuestra máquina Kali y lo descargaremos en la máquina víctima utilizando **wget**, para luego moverlo a la carpeta **/opt**.    

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/chocolate_lovers/C_18.jpg)    

Ahora solo deberemos ponernos a la escucha en nuestra máquina victima para recibir la reverse shell como root.   

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/chocolate_lovers/C_19.jpg)      

















