# Domain

# Enumeración

Empezamos con un escaneo de los puertos.

`sudo nmap -p- --open -sC -sS -sV --min-rate=5000 -n -Pn -vvv 172.17.0.2 -oN Domain`  

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

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Domain/D_1.jpg)     

Tenemos 3 puertos abiertos: 80, 139 y 445.  
Lo primero que haremos será intentar enumerar SMB utilizando una sesión anónima, ejecutando el comando:  

`rpcclient 172.17.0.2 -U "" -N`  

Una vez dentro, podemos intentar enumerar los usuarios utilizando el comando `enumdomusers`.  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Domain/D_2.jpg)    

Bien, sabemos que en la máquina hay dos usuarios: **bob** y **james**. 
Por lo tanto, realizaremos un ataque de fuerza bruta contra SMB utilizando la herramienta **netexec**.  
Primero, crearemos una lista llamada **users.txt** que contenga los usuarios recién descubiertos y luego utilizaremos el diccionario **rockyou.txt** para intentar encontrar credenciales válidas.  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Domain/D_3.jpg)    

Después de esperar unos momentos, obtenemos credenciales válidas.  
Ahora intentaremos enumerar los recursos a los que tiene acceso el usuario **bob**.  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Domain/D_4.jpg)     

Como podemos observar, tenemos permisos de lectura y escritura en la carpeta **html**, que es probablemente donde está alojado el sitio web.  
Lo que podemos hacer es conectarnos con **smbclient** y cargar una reverse shell en PHP.  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Domain/D_5.jpg)    

Ahora solo nos queda ponernos en escucha en nuestra máquina Kali y visitar la página web.  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Domain/D_6.jpg)    

¡Perfecto!  
¡Ya estamos dentro!  

# Escalada de privilegios  

Una vez dentro, como de costumbre, estabilizaremos la shell para poder trabajar de manera más cómoda.  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Domain/D_7.jpg)    

Ahora ejecutaremos los comandos clásicos para identificar cómo escalar a root.  
Con `sudo -l` no obtenemos nada.  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Domain/D_8.jpg)      

Ejecutamos el comando `find / -perm -4000` y aquí sí, nos llama la atención el binario **nano**.  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Domain/D_9.jpg)   

Lo que podríamos hacer es aprovechar este permiso SUID en **nano** para modificar el archivo **/etc/passwd** y eliminar la contraseña de root.  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Domain/D_10.jpg)   

Al modificarlo de esta manera, ahora solo nos quedará escribir `su` y...

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Domain/D_11.jpg)   

¡Listo!  
¡Ya somos root!  


 
