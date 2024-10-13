# Move

# Enumeración

Empezamos con un escaneo de los puertos.

`sudo nmap -p- --open -sC -sS -sV --min-rate=5000 -n -Pn -vvv 172.17.0.2 -oN Move`  

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

![M](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/move/M_1.jpg)   

Tenemos 4 puertos abiertos, el 21, 22, 80 y 3000.  
Echamos un ojo al puerto 80, pero no vemos nada interesante, tampoco en el código fuente.  

![M](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/move/M_2.jpg)    

Hacemos un podo de fuzzin y descubimos una pagina que se llama **maintenance.html**.  

`sudo gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-large-directories-lowercase.txt -x bak,htm,html,php,txt`  

![M](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/move/M_3.jpg)    

La abrimos y vemos que hay una posible pista.  

![M](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/move/M_4.jpg)    

Miramos en el puerto 3000 para ver lo que hay y vemos que hay un panel de login de grafana.   

![M](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/move/M_5.jpg)    

Miramos en Google cuales son las credenciales por defecto.  

![M](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/move/M_6.jpg)     

No logramos acceder.   
Lo que podemos apreciar es que nos encontramos en frente a la versión **8.3.0**.     
Por lo tanto miramos a ver si hay algunas vulnerabilidades con la herramienta Metasploit. 

`searchsploit grafana`

![M](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/move/M_7.jpg)    

Nos copiamos en local el script python con el comando `searchsploit -m  50581`.  

![M](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/move/M_8.jpg)      

Lo ejecutamos pero vemos que al parecer no nos funciona.  

![M](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/move/M_9.jpg)     

Despues de una búsqueda en Google encontramos otro script en python que sí que nos sirve.  
Lo ejecutamos e intentamos leer el fichero /etc/passwd.  

![M](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/move/M_10.jpg)       

Como podemos ver hay un usuario nombrado **freddy**.  
Ahora intentamos leer el fichero /tmp/pass.txt que habíamos descubierto anteriormente.  

![M](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/move/M_11.jpg)    

Encontramos una probable contraseña. 
Probamos a ver si vale para el usuario freddy.  

![M](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/move/M_12.jpg)      

Perfecto, ya estamos dentro.  
Ahora probamos con el comando `sudo -l` para ver si podemos escalar privilegios.  

![M](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/move/M_13.jpg)     

Lo que haremos será ver si podemos editar el fichero maintenance.py. 
Buscamos uno script en python para enviarnos una rever shell a nuestra máquina kali y editamos el fichero con el comando `nano /opt/maintenance.py`.  

![M](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/move/M_14.jpg). 

Ahora nos pondremos a la escucha en nuestra máquina kali con el comando `sudo nc -lvnp 6969`.  
Por fin ejecutaremos en la máquina víctima el comando `sudo -u root /usr/bin/python3 /opt/maintenance.py` y listo!  
Ya somos root!  

![M](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/move/M_15.jpg). 





