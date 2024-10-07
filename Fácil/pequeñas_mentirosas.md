# Pequeñas mentirosas

# Enumeración

Empezamos con un escaneo de los puertos.

`sudo nmap -p- --open -sC -sS -sV --min-rate=5000 -n -Pn -vvv 172.17.0.2 -oN PequeñasMentirosas`  

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

![PM](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/peque%C3%B1as_mentirosas/PM_1.jpg) 

Tenemos 2 puertos abiertos, el 22 y el 80. 
Echamos un ojo a lo que corre en el puerto 80.  

![PM](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/peque%C3%B1as_mentirosas/PM_2.jpg)  

Vale, intentamos hacer un poco de fuzzing web para ver si podemos sacar algo util.  
Para ello le damos al comamdo:  

`sudo gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt`  

![PM](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/peque%C3%B1as_mentirosas/PM_3.jpg)      

Bueno, no sacamos nada de nada.  
Trás razonar un rato, volvemos a analizar la web, y probamos a ver si **a** es el nobre de un posible usuario.  
Intentaremos hacer un ataque de fuerza bruta contra ssh.  
Ejecutaremos el comando:  

`sudo hydra -l a -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2`  

![PM](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/peque%C3%B1as_mentirosas/PM_4.jpg)   

Listo! 
Ya tenemos un usuario y contraseña valido. 
Entramos por ssh.  
Una vez dentro probamos los tipicos comandos para intentar escalar de privilegios:  

`sudo -l`  

`find / -perm -4000 2>/dev/null`  

![PM](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/peque%C3%B1as_mentirosas/PM_5.jpg)     

Vale, deberemos ver si hay otra forma. 
Miramos a ver si hay otros usuarios en la máquina.  
Le damos a `cat /etc/passwd` y vemos que nos encontra otro usuario nombrado **spencer**.  

![PM](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/peque%C3%B1as_mentirosas/PM_6.jpg)       

Intentaremos otra vez a realizar un ataque de fuerza bruta contra ssh.  

`sudo hydra -l spences -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2`    

![PM](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/peque%C3%B1as_mentirosas/PM_7.jpg)        

Perfecto! 
Ahora cambiaremos de usuario con el comando `su spencer`.  

![PM](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/peque%C3%B1as_mentirosas/PM_8.jpg)       

Ahora intentamos escalar privilegios para convertirnos en root.  
Le damos al comando `sudo -l`  

![PM](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/peque%C3%B1as_mentirosas/PM_9.jpg)    



