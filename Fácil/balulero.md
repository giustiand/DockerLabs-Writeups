# Balulero
**Herramientas y recursos usados**  
- zip2john
- john the ripper

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

Tenemos dos puertos abiertos, el 22 y el 80.  
Echamos un vistazo a la web a ver si hay algo interesante.   

![B](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/balulero/B_2.jpg)    

A primera vista no hay nada interesante.  
Miramos si hay algo en el código fuente.  

![B](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/balulero/B_3.jpg)   

Como podemos ver hay uno **script.js**, probamos a ver si hay algo interesante por allí.   
Efectivamente hay un comentario que nos debería llamar a la atencíon.  

![B](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/balulero/B_4.jpg)    

Mirmaos enteonces la página .env_de_baluchingon  

![B](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/balulero/B_5.jpg)      

Perfecto! 
Ya tenemos un nombre de usuario y una contraseña, por lo tanto podemos probar a hacer login atráves de ssh.  

# Explotación y escalada de privilegios  

Perfecto! 
Ya tenemos un nombre de usuario y una contraseña, por lo tanto podemos probar a hacer login atráves de ssh.  

![B](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/balulero/B_6.jpg)   

Listo!  
Estamos dentro!   
Si damos el comando `sudo -l` podemos ver que podemos ejecutar, con el usuario **chocolate**, el binario php sin necesitar una password.  
Por lo tanto mirammos en GTFOBins si podemos encontrar algo que podemos utilizar.  

![B](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/balulero/B_7.jpg)    

Ahora seguimos los pasos indicados y deberáimos poder escalar al usuario chocolate.  

![B](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/balulero/B_8.jpg)      

Una vez dentro con el usuario chocolate tenenmos que intentar a escalar a root.  
Con los comandos `sudo -l` y ``find / -perm -4000 2>/dev/null` no encontramos nada interesante.  

![B](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/balulero/B_9.jpg)   

Por lo tanto lo que se me ocurre es subir a la maquina victima el archivo **pspy64** para ver si hay algun proceso de que podemos abusar.  
Entonces me montaré un servidor web en python en mi kali y descargaré el fichero en la maquina victima.  

![B](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/balulero/B_10.jpg)     

![B](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/balulero/B_11.jpg)     

Le damos los permisos de ejecucíon y lo lanzamos.  

![B](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/balulero/B_12.jpg)    

Como podemos ver hay uno **script.php** que se ejecuta de forma recurrente.  
Miramos a ver que contiene el script.  
