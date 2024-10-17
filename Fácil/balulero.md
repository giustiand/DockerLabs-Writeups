# Balulero

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

![B](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/balulero/B_1.jpg)   

Tenemos dos puertos abiertos: el 22 y el 80.   
Echamos un vistazo a la web para ver si hay algo interesante.  

![B](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/balulero/B_2.jpg)    

A primera vista, no hay nada interesante.   
Revisamos si hay algo en el código fuente.  

![B](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/balulero/B_3.jpg)   

Como podemos ver, hay un **script.js**. Probamos a ver si hay algo interesante allí.    
Efectivamente, hay un comentario que debería llamarnos la atención.  

![B](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/balulero/B_4.jpg)    

Veamos entonces la página **.env_de_baluchingon**.  

![B](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/balulero/B_5.jpg)      

¡Perfecto!    
Ya tenemos un nombre de usuario y una contraseña, por lo tanto, podemos intentar iniciar sesión a través de SSH.  

# Explotación y escalada de privilegios  

¡Perfecto!  
Ya tenemos un nombre de usuario y una contraseña, por lo tanto, podemos intentar iniciar sesión a través de SSH.  

![B](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/balulero/B_6.jpg)   

¡Listo!  
Estamos dentro.  
Si ejecutamos el comando `sudo -l`, podemos ver que podemos ejecutar, con el usuario **chocolate**, el binario **php** sin necesidad de una contraseña.  
Por lo tanto, consultamos GTFOBins para ver si encontramos algo que podamos utilizar.  

![B](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/balulero/B_7.jpg)    

Ahora seguimos los pasos indicados y deberíamos poder escalar al usuario **chocolate**.  

![B](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/balulero/B_8.jpg)      

Una vez dentro con el usuario **chocolate**, tenemos que intentar escalar a root.    
Con los comandos `sudo -l` y `find / -perm -4000 2>/dev/null`, no encontramos nada interesante.  

![B](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/balulero/B_9.jpg)   

Por lo tanto, se me ocurre subir a la máquina víctima el archivo **pspy64** para ver si hay algún proceso del que podamos abusar.    
Entonces, montaré un servidor web en Python en mi Kali y descargaré el archivo en la máquina víctima.  

![B](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/balulero/B_10.jpg)     

![B](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/balulero/B_11.jpg)     

Le damos permisos de ejecución y lo lanzamos.  

![B](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/balulero/B_12.jpg)    

Como podemos ver, hay un **script.php** que se ejecuta de forma recurrente.  

![B](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/balulero/B_13.jpg)   

Revisamos qué contiene el script.  

![B](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/balulero/B_14.jpg)     

Dado que tenemos permisos para borrar el archivo, se me ocurre subir una reverse shell en PHP a la carpeta **/tmp** de la máquina víctima.   
Por lo tanto, montaré nuevamente el servidor en Python y subiré la reverse shell (podemos consultar la web de Pentest Monkey).  

Una vez modificado el archivo y subido a la máquina víctima, lo que tenemos que hacer es borrar el archivo **script.php** que se encuentra en **/opt** y sustituirlo con nuestro archivo que contiene la reverse shell, asegurándonos de nombrarlo siempre como **script.php**.  

![B](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/balulero/B_15.jpg)       

Ahora solo tendremos que abrir una shell en nuestra máquina Kali y ponernos a la escucha, y en unos segundos... ¡listo!    
¡Somos root!  

![B](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/balulero/B_16.jpg)         







